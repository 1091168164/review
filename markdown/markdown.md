###markdown的使用


######标题
# 一级标签   

<h1>一级标签</h1> 

## 二级标签   

<h2>二级标签</h2> 

### 三级标题

<h3>三级标签</h3> 

#### 四级标题

<h4>四级标签</h4> 

##### 五级标题

<h5>五级标签</h5> 

###### 六级标题

<h6>六级标签</h6> 


 一级标签  
 =======
 
 二级标签
 ------
 
 ***
 
 ###### 列表
 
 ###### 无序列表
 * 1                  
 + 1           
 - 1    
 
 ***
 分割线(粗细)
 ----
 
 <ul>
   <li>1</li>
   <li>1</li>
   <li>1</li>
 </ul>
 
 *** 
 
 ###### 有序列表
  
  1.1                  
  2.1           
  3.1 
  
  ---
  <ol>
    <li>1</li>
    <li>1</li>
    <li>1</li>
  </ol>
 
 ***
  ###### 引用
 >引用 这是一段代码引用
 ***
 
   ###### 链接
   
   >2种链接方式：行内式和参数式，链接文字用 [链接文字]标记 
   
   ###行内式
 [Windows/Mac/Linux 全平台客户端](https://www.zybuluo.com/cmd/)
 <p><a href="https://www.zybuluo.com/cmd/">Windows/Mac/Linux 全平台客户端</a></p>

----

###参数式
[Windows/Mac/Linux 全平台客户端](https://www.zybuluo.com/cmd/ 'title属性')

<p><a href="https://www.zybuluo.com/cmd/" title="黑色">Windows/Mac/Linux 全平台客户端</a></p>

---

### 图片
![cmd-markdown-logo](https://www.zybuluo.com/static/img/logo.png)

<p><img src="https://www.zybuluo.com/static/img/logo.png" alt="cmd-markdown-logo" title="" /></p>

----

#####代码段

######单行使用``【反引号】
`我是单行文本`

######多行请注意
```
`第一行`
`第二行`
`第三行`
```

---

```java
    /** 
    * @Description:  
    * @Author: tangx.w 
    * @Date: 2020/7/15 上午11:50
    */ 
    public static void main(String[] args){
      System.out.println(a+a);
    }
```

---

| 项目        | 价格    |  数量   |
| --------    | -----: | :----:  |
| 计算机      | \$1600  |   5    |
| 手机        |   \$12  |   12   |
| 管线        |    \$1  |   234  |

: 是对齐方向
