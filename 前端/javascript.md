# 基础知识
## 零碎知识点:
修改了style再操作class里的样式是无效的（优先级）
可以自定义属性如a.index[i]=i
可以通过改变className属性的值改变样式
### 关于等号
- a==b 先转换类型再比较
- a===b 不转换直接比较

### 隐藏显示
- style.dispay='none'不显示
- 'block'显示

### 计时器
```javasctipt
//启动计时器，调用overs函数
var interval = setInterval(overs,1000);
//停止计时器
clearInterval(interval);
interval = null;
```
### Splice--数组操作
[w3cschool](https://www.w3school.com.cn/jsref/jsref_splice.asp "w3cschool")
## 代码部分知识点：
```javascript
//操作属性的方法
function setStuff(stuff,value){
	var a=document.getElementById('A');
	a[stuff]=value;
};
```
```javascript
//省去起名字的方法
a.onclick=function(){  ... };
```
```javascript
//一组操作（具体用数组a.Length）
var a=oDiv.getElementsByTagName('div');
```
```javascript
//arry.sort([比较函数])
arr.sort(function(n1,n2){
    return n1-n2;
});
```
