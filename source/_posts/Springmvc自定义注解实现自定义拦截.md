
title: "Springmvc自定义注解实现自定义拦截"
date: 2016-07-11  23:46:49
tags: [自定义注解,springmvc]
categories: java
---

我自己搭的springmvc的项目，想添加登陆校验。
最简易优雅的实现，在需要用户信息才可以操作的Controller的方法上面加上一个@CheckLogin就可以实现登陆校验。
假如未登录–>跳转到登录页–>登陆成功–>自动跳回刚才要执行的动作方法。

用springmvc的handlerinterceptor的来实现。

## 一.首先介绍一下action拦截器：
HandlerInterceptor是Spring MVC为我们提供的拦截器接口，来让我们实现自己的处理逻辑，HandlerInterceptor 的内容如下：

``` aspectj
public interface HandlerInterceptor {
    boolean preHandle(
            HttpServletRequest request, HttpServletResponse response,
            Object handler)
            throws Exception;

void postHandle(  
        HttpServletRequest request, HttpServletResponse response,   
        Object handler, ModelAndView modelAndView)   
        throws Exception;  

void afterCompletion(  
        HttpServletRequest request, HttpServletResponse response,   
        Object handler, Exception ex)  
        throws Exception;  
}
```


可以看到接口有3个方法，其含义如下：

> preHandle：在执行action里面的处理逻辑之前执行，它返回的是boolean，这里如果我们返回true在接着执行postHandle和afterCompletion，如果我们返回false则中断执行。
> 
> postHandle：在执行action里面的逻辑后返回视图之前执行。
> 
> afterCompletion：在action返回视图后执行。
> 
> HandlerInterceptorAdapter适配器是Spring
> MVC为了方便我们使用HandlerInterceptor而对HandlerInterceptor
> 的默认实现，里面的3个方法没有做任何处理，在preHandle方法直接返回true，这样我们继承HandlerInterceptorAdapter后只需要实现3个方法中我们需要的方法即可，而不像继承HandlerInterceptor一样不管是否需要3个方法都要实现。
> 
> 当然借助于HandlerInterceptor我们可以实现很多其它功能，比如日志记录、请求处理时间分析等，权限验证只是其中之一。

<!--more-->

## 二.下面我们就来一步一步来完成注解式权限验证的功能。
首先添加一个账户的Controller和登录的Action及视图来模拟在没有权限时跳转到登陆页面，内容分别如下：

### 1.新建包com.bruce.interceptor包,添加自定义注解CheckLogin.java，内容如下：

``` java
package com.bruce.interceptor;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
登陆校验|此注解写在用于登录权限的Controller上面
@author bruce
@date 2016年7月11日
@email zhangyang226@gmail.com
@site http://blog.northpark.cn | http://northpark.cn | orginazation https://github.com/jellyband
 * 
 */
@Documented
@Inherited
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CheckLogin {

boolean validate() default true;
}
```


### 2.添加自己的拦截器实现CheckLogin继承于HandlerInterceptorAdapter，

com.bruce.interceptor包中的LoginInterceptor.java 内容如下：

``` aspectj
package com.bruce.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import com.bruce.constant.BC\_Constant;
import com.bruce.model.User;

/**
登陆拦截器.
@author zhangyang
 *
 */
public class LoginInterceptor extends HandlerInterceptorAdapter {

@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {


    if(handler.getClass().isAssignableFrom(HandlerMethod.class)){
        CheckLogin checklogin = ((HandlerMethod) handler).getMethodAnnotation(CheckLogin.class);

        //没有声明需要权限,或者声明不验证权限
        if(checklogin == null || checklogin.validate() == false){
            return true;
        }else{                
            //在这里实现自己的权限验证逻辑
            User user = (User) request.getSession().getAttribute("user");
            if(user!=null){//如果验证成功返回true（这里直接写false来模拟验证失败的处理）
                return true;
            }else{//如果验证失败
                //返回到登录界面
                String url = request.getRequestURL().toString();
                String[] strs = url.split("8082/");
                String postfix = strs[1];
                url = "http://"+BC_Constant.Domain+"/"+postfix;
                response.sendRedirect("/login?redirectURI="+url);
                return false;
            }       
        }
    }else{
        return true;   
    }


  }



@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    super.postHandle(request, response, handler, modelAndView);
}
}
```


### 3.配置项目的spring-mvc.xml添加如下内容：

``` vbscript-html
 <!-- 定义拦截器 -->
<mvc:interceptors>  
    <!-- 国际化操作拦截器 如果采用基于（请求/Session/Cookie）则必需配置 --> 
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />  
    <!-- 如果不定义 mvc:mapping path 将拦截所有的URL请求 -->
    <bean class="com.bruce.interceptor.LoginInterceptor"></bean>
</mvc:interceptors>
```


### 4.具体的Controller写法

``` nimrod
//添加次注解，未登录的自动跳转到登录页
@CheckLogin
@RequestMapping("/add")
public String toAdd(ModelMap map,String userid,HttpServletRequest request,HttpServletResponse response) {
String result = "/page/user/lyricAdd";
return result;
}    
```


可以看到正确执行了权限判断逻辑，这样我们只需要在我们在需要权限验证的action上加上这个注解就可以实现权限控制功能了。

注解式权限验证的内容到此结束。