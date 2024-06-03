# Explaining Variables
```java
return new Rectangle(
        new Point(0, 0),
        new Point(5, 0),
        new Point(0, 5),
        new Point(5, 5)        
);
```

* 각각의 값이 뭘 의미하는지 설명하는 변수로 만들자.

```java
Point leftTop = new Point(0, 0),
Point rightTop = new Point(5, 0),
Point leftBottom = new Point(0, 5),
Point rightBottom = new Point(5, 5)

return new Rectangle(leftTop, rightTop, leftBottom, rightBottom);
```