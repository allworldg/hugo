---
title: "Ruoyi Dict"
draft : true
date : 2022-10-07T10:21:04+08:00
description : ""
slug : "" 
tags : [ruoyi,dict,字典]
categories : [ruoyi]
externalLink : ""
---
此文章属于[ruoyi项目实战](https://allworldg.xyz/tags/ruoyi/)系列

## 使用目的
1. 什么是字典数据：具体的值（0，1，"Y"，"N")，对应具体的业务逻辑（"男"，"女"，"是"，"否"）。
2. 字典数据不应该只写死在代码中，还应存入数据库，通过管理系统来增删改查。

## 源码分析
ruoyi项目在低于3.7.0的版本中，前端字典功能实现比较简单，每个index.vue页面都请求dict的api，获取数据再加工显示即可。3.7.0之后的版本使用了混入，所以复杂了一些。

## 分析
1. 入口：查看全局入口文件`main.js`，`DictData.install()`是字典功能的入口位置。
	```Javascript
	function install() {
	  Vue.use(DataDict, {//额外参数
	    metas: {
	      '*': {
	        labelField: 'dictLabel',
	        valueField: 'dictValue',
	        request(dictMeta) {
	          return getDicts(dictMeta.type).then(res => res.data)
	        },
	      },
	    },
	  })
	}
	```
	install全局注册了一个插件`DataDict`，同时传入了额外参数`{meta:xxx}`，目的是将DataDict插件对应的参数进行赋值。

2. DataDict插件：因为该插件本身是个function，所以Vue.use会直接将function视为`install()`方法执行。
	```JavaScript
	export default function (Vue, options) {
		mergeOptions(options)
		Vue.mixin({...})
	}
	```
	首先执行`mergeOptions(options)`,目的是将传入的额外参数与DictOptions合并。具体实现是通过递归调用`mergeRecursive(source,target)`，将DictOptions的属性覆盖或者添加。

	其次注册全局混入 `Vue.mixin` ，给所有 Vue 实例添加了 `data()` 和 `created()` 方法。
	```JavaScript
	Vue.mixin({
		data(){
			const dict = new Dict()
			dict.owner = this
			return {dict}
		},
		created(){
		....
		this.dict.init(this.$options.dicts).then(()=>{...})
		}
		
	})
	```
	data (): 每个 Vue 页面创建一个 Dict。
	
	created(): 调用Dict.init(dicts)方法，传入每个vue页面声明的dicts数组(例如 `dicts['sys_normal_disable']`)。(额外补充：init().then(....)里的方法个人认为是为了拓展性，因为我全局查找也没有看到任何地方用到。)

3. Dict. init () : 看注释即可
	```JavaScript
	init(options) {  
	  if (options instanceof Array) {  
	    //此处传进来的是每个index.vue的dicts属性，基本上是['dictName1','dictName2']之类的。  
	    options = {types: options}  
	  }  
		  const opts = mergeRecursive(DEFAULT_DICT_OPTIONS, options)//options与DEFAULT合并，并且将合并结果赋值给opts  
	  if (opts.types === undefined) {  
	    throw new Error('need dict types')  
	  }  
	  const ps = []  
	  this._dictMetas = opts.types.map(t => DictMeta.parse(t)) //调用parse,将数组中的字符串转换为DictMeta对象返回。 
	  this._dictMetas.forEach(dictMeta => {  
	    const type = dictMeta.type  
	    Vue.set(this.label, type, {})//dict.label添加属性 dictName:{}
	    Vue.set(this.type, type, [])//dict.type 添加属性 dictName[]
	    if (dictMeta.lazy) {  
	      return  
	    }  
	    ps.push(loadDict(this, dictMeta))  
	  })loadDict:请求后端api，将数据组装进dict  
	  return Promise.all(ps)  
	}
	```

简单通过注释解释一下init里的一些调用函数源码
4. DictMeta.parse
	```JavaScript
	DictMeta.parse= function(options) {  
	  let opts = null  
	  if (typeof options === 'string') {  
	    opts = DictOptions.metas[options] || {}  
	    opts.type = options//opt{type:'字典名称'}  
	  } else if (typeof options === 'object') {  
	    opts = options  
	  }  
	  //创建{type:'字典名称"}并且赋值给DictOptions.meta属性  
	  opts = mergeRecursive(DictOptions.metas['*'], opts)  
	  //构造dictmeta原数据  
	  return new DictMeta(opts)  
	}
	```
	主要将vue页面的dicts数组以及DictOption的meta数据在整合赋值到DictMeta对象，方便后续调用。

5. loadDict(dict,dictMeta)
	```JavaScript
	function loadDict(dict, dictMeta) {  
	  return dictMeta.request(dictMeta)//请求后端api，获取字典数据  
	    .then(response => {  
	      const type = dictMeta.type  
	      let dicts = dictMeta.responseConverter(response, dictMeta)//将response转换成DictData  
	      if (!(dicts instanceof Array)) {  
	        console.error('the return of responseConverter must be Array.<DictData>')  
	        dicts = []  
	      } else if (dicts.filter(d => d instanceof DictData).length !==
	       dicts.length) {  
	        console.error('the type of elements in dicts must be DictData')  
	        dicts = []  
	      }  
	      //将response的数据插入到dict.type['dictName']的数组中  
	      //splice实现了响应式改变数组元素，所以这里不用vue.set  
	      dict.type[type].splice(0, Number.MAX_SAFE_INTEGER, ...dicts)
	      //将dicts(也就是dictData)赋值给dict.type[type]  
	      dicts.forEach(d => {  
	        Vue.set(dict.label[type], d.value, d.label)
	        //dict.label{'dictName':{}}添加属性d.value:d.label  
	      })  
	      return dicts  
	    })  
	}	
	```

6. 具体页面应用
	例如job/index.vue,
	```JavaScript
	<el-select v-model="queryParams.status" placeholder="请选择任务状态" clearable>  
	  <el-option  
	    v-for="dict in dict.type.sys_job_status"  
	    :key="dict.value"  
	    :label="dict.label"  
	    :value="dict.value"  
	  />  
	</el-select>	

	export default{
	dicts:['sys_job_group','sys_job_status'],
	//dict:{'sys_job_group':[data1,data2],'sys_job_status':[data1,data2]} 通过上文的代码全局混入得到
	}
	```
