[toc]

---

# Bean 作用域

1. singleton(单例)：Spring容器只会创建一个bean对象；
2. prototype：每次获取bean都会重新创建一个bean对象；
3. request：对于每一个http请求，在同一个请求内Spring容器只会创建一个bean对象，若请求结束，bean也随之销毁；
4. session：在同一个http会话里，Spring容器只会创建一个bean对象，若传话结束，也随之销毁；
5. globalSession：globalSession作用域的效果与session作用域类似，但是只适用于基于portlet的web应用程序中
6. application：在servlet程序中，该作用域的bean将会作为ServletContext对象的属性，被全局访问，与singleton的区别就是，singleton作用域的bean在Spring容器中只一；application作用域的bean在ServletContex中唯一；
7. websocket：为每个websocket对象创建一个实例。仅在Web相关的ApplicationContext中生效。