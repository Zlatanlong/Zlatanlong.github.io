---
title: (转)java List 排序 Collections.sort()
date: 2018-10-7
updated: 2018-10-7
tags: 
  - 排序
categories: java
---
### 用Collections.sort方法对list排序有两种方法第一种是list中的对象实现Comparable接口，如下：
<!-- more -->
```java
/**
 * 根据order对User排序
*/
public class User implements Comparable<User>{
     private String name;
     private Integer order;
     public String getName() {
         return name;
     }
     public void setName(String name) {
         this.name = name;
     }
     public Integer getOrder() {
         return order;
     }
     public void setOrder(Integer order) {
         this.order = order;
     }
     public int compareTo(User arg0) {
         return this.getOrder().compareTo(arg0.getOrder());
     }
 } 测试一下：
public class Test{
 
     public static void main(String[] args) {
         User user1 = new User();
         user1.setName("a");
         user1.setOrder(1);
         User user2 = new User();
         user2.setName("b");
         user2.setOrder(2);
         List<User> list = new ArrayList<User>();
         //此处add user2再add user1
        list.add(user2);
         list.add(user1);
         Collections.sort(list);
         for(User u : list){
             System.out.println(u.getName());
         }
     }
 } 
```
输出结果如下
a

b
### 第二种方法是根据Collections.sort重载方法来实现，例如：
```java
/**
 * 根据order对User排序
*/
public class User { //此处无需实现Comparable接口
    private String name;
     private Integer order;
     public String getName() {
         return name;
     }
     public void setName(String name) {
         this.name = name;
     }
     public Integer getOrder() {
         return order;
     }
     public void setOrder(Integer order) {
         this.order = order;
     }
 }
 
 主类中这样写即可(HastSet——>List——>sort进行排序)：
public class Test {
    public static void main(String[] args) {
        User user1 = new User();
        user1.setName("a");
        user1.setPrice(11);
        User user2 = new User();
        user2.setName("b");
        user2.setPrice(2);
 
        Set<User> Hset = new HashSet<User>();
        Hset.add(user2);
        Hset.add(user1);
 
        List<User> list = new ArrayList<User>();
        list.addAll(Hset);
 
 
        Collections.sort(list,new Comparator<User>(){
            public int compare(User arg0, User arg1) {
                return arg0.getPrice().compareTo(arg1.getPrice());
            }
        });
        for(User u : list){
            System.out.println(u.getName());
        }
    }
```
输出结果如下：

a
b

默认为升序，将。return arg0.getOrder().compareTo(arg1.getOrder()); 改为：
return arg1.getOrder().compareTo(arg0.getOrder());   
就成降序的了。

> 转载自: [csdn zxy_snow](https://me.csdn.net/zxy_snow)