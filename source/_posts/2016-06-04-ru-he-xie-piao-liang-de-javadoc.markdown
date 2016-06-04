---
layout: post
title: "如何写漂亮的JavaDoc"
date: 2016-06-04 16:08:14 +0800
comments: true
categories: tools
---
##### 版权声明：本文所有内容都是原创，如有转发请注明出处，谢谢。

注释在编程中，非常的重要。一个好的注释，是我们有好的编程习惯的基础。JavaDoc很简单，只要稍微注意几个点，我们就可以写出很好看的doc。这样我们的编程能力，至少看起来会好很多。

### 一个简单的例子: 

```
/**
 * Returns an Image object that can then be painted on the screen. 
 * The url argument must specify an absolute {@link URL}. The name
 * argument is a specifier that is relative to the url argument. 
 * <p>
 * This method always returns immediately, whether or not the 
 * image exists. When this applet attempts to draw the image on
 * the screen, the data will be loaded. The graphics primitives 
 * that draw the image will incrementally paint on the screen. 
 *
 * @param  url  an absolute URL giving the base location of the image
 * @param  name the location of the image, relative to the url argument
 * @return      the image at the specified URL
 * @see         Image
 */
 public Image getImage(URL url, String name) {
        try {
            return getImage(new URL(url, name));
        } catch (MalformedURLException e) {
            return null;
        }
 }
```
From this [How To Wrte JavaDoc](http://www.oracle.com/technetwork/articles/java/index-137868.html)。几个比较重要的点有：
### 段落注释
```
段落分行: <p>
 
列表: 
<ul>
  <li> point 1
  <li> point 2
</ul> 

分行: <br>

插入代码:
Class Usage: 
<code> 
 import org.apache.Example
</code>
```

### 参数注释
```
@author (classes and interfaces only, required)        
@version (classes and interfaces only, required.   
@param (methods and constructors only)        
@return (methods only)        
@exception (@throws is a synonym added in Javadoc 1.2)        
@see  
@since  
@serial (or @serialField or @serialData)        
@deprecated 
```

### 链接注释 用于@SEE 
```
@see #field 
@see #Constructor(Type, Type...) 
@see #Constructor(Type id, Type id...) 
@see #method(Type, Type,...) 
@see #method(Type id, Type, id...) 
@see Class 
@see Class#field
@see Class#Constructor(Type, Type...) 
@see Class#Constructor(Type id, Type id) 
@see Class#method(Type, Type,...) 
@see Class#method(Type id, Type id,...) 
@see package.Class 
@see package.Class#field 
@see package.Class#Constructor(Type, Type...) 
@see package.Class#Constructor(Type id, Type id) 
@see package.Class#method(Type, Type,...) 
@see package.Class#method(Type id, Type, id) 
@see package 
```
