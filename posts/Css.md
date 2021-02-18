---
title: 'CSS基础'
date: '2018-10-05 22:47:54'
---
### 布局
#### input标签和span标签一行显示
盒子宽度已知，要求span标签和input标签加起来的宽度等于盒子的宽度
<!--more-->
1. 给span标签和input标签严格定义宽度，如span 30，input 70，
2. 不要span标签，使用伪类：before，使用绝对布局，input设置相应的marginLeft值
```jsx harmony
// jsx
 <div style={Styles.item} className=" infoItem userName" onClick={()=>this.onClick({type:'userName',com:this.userNameInputCom})}>
      <input ref={com=>this.userNameInputCom=com} onChange={()=>this.userNameChange({userNameCom:this.userNameInputCom,passwordCom:this.passwordInputCom})} />
 </div>
 
 const Styles = {
     wrap: {
         paddingTop: 30,
         paddingBottom: 30,
         paddingLeft: 40,
         paddingRight: 40,
         display:'inline-block'
     },
     item:{
         borderBottom:'1px solid lightgray',
         position:'relative',
         paddingTop:'1.5rem',
         paddingLeft:'3.2rem',
         paddingBottom:'.5rem'
     }
 };
```
```css
 /*css*/
 .infoItem:before{
    content:'';
    position:absolute;
    left:1rem;
    top:1.5rem;
 }
 .userName:before{
    content:'账号';
 }
  .password:before{
   content:'密码';
  }
```

#### 盒子高度未知，盒子垂直居中
目前没有好的方法，使用chrome开发者工具查看盒子的高度，用js获取屏幕当前的高度减去盒子高度的一半作为盒子的上外边距，或者干脆就是用百分比估计一个一个的试。

2018-10-08 补充

自适应垂直居中,绝对定位+auto，效果图：

![img](http://pga6xqrjk.bkt.clouddn.com/blog/blog_jietu1.png)
```css
 img.portrait{
     position:absolute;
     left:0;
     right:0;
     top:0;
     bottom:0;
     margin:auto;
 }
 /*或者*/
  img.portrait{
      position:absolute;
      left:50%;
      top:50%;
      transform:translate(-50%,-50%);//本身的一半
  }
```

#### 蒙层
定义一个同级的空盒子，使用绝对布局，上下左右边距为0，设置透明度，调整zIndex值
```jsx 
 mask: {
        position: 'absolute',
        top: '0',
        left: '0',
        right: '0',
        bottom: '0',
        background: 'rgba(0, 0, 0, 0.6)',
        zIndex: '2',
    },
```
