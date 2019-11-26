---
title : Python 기초
category : [Python]
tags : [Python]
date : 2019-11-26T23:02:00Z
comments : true
author_profile: false
---

- Data types
    - 기본 데이터 타입   
        <pre>
        # 변수의 표현
        Java and JavaScript = CamelCase
        Python = Snake_Case
        
        print(type(a_none))
        
        a_string = '2'
        a_number = 3
        a_boolean = True
        a_float = 3.14
        a_none = None
        </pre>
        
    - 열거형 데이터 타입(sequence type)
        <pre>
        # list (implement common and mutable) => mutable
        days = ["Mon", "Tue", "Wed", "Thur", "Fri"]
        dayas.append("Sun")
        days.reverse()
       
        # tuple (implement common) => immutable
        days = ("Mon", "Tue", "Wed", "Thur", "Fri")
        </pre>
    - 맵핑 타입
        <pre>
        # dictionary(key and value)
        nico = {
            "name" : "Nico",
            "age" : 12,
            "Korean" : True,
            "fav_food" : ["Pizza","Kimchi"]
        }
        
        print(nico["age"])
        print(nico)
        # add
        nico["hansome"] = True
        print(nico)
        </pre>
        
- 함수(function)
    > 들여쓰기를 통해 함수의 바디를 작성한다.
    - positional arguments
        <pre>
        def say_hello(who):
            print("hello", who)
            
        def plus(a, b):
            print(a + b)
        </pre>
        
    - keyword arguments
        <pre>
        # 매개변수의 이름과 값을 명시적으로 표현해준다.
        def say_hello(name, age):
            return f"hello {name} you are {age} years old"
        hello = say_hello('nico', '12')
        hello = say_hello(age = '12', name='nico')
        </pre>
        
- else if , elif
    <pre>
    def plus(a, b):
        if type(b) is int or type(b) is float:
            return a + b
        else:
            return None
            
    def age_check(age):
        print(f"you are {age}")
        if age < 18:
            print('you can't drink)
        elif age = 18:
            print('you are new to this')
        else:
            print('enjoy your drink')
    
    age_check(23)
    </pre>
    
- for in
    <pre>
    days = ("Mon", "Tue", "Wed", "Thr", "Fri")
    
    for x in days:
        if x is "Wed":
            break
        else:
            print(x)
            
    # result is Mon Tue
    </pre>
    
- module import
    <pre>
       # math 모듈 전부 import
       import math
       
       # 특정 함수 import
       from math import ceil, fsum
       print(ceil(1.2))
       print(fsum([1,2,3,4,5,6]))
    </pre>
    
- Django의 특징
    > 장고는 자바의 스프링과 같은 프레임워크이다.  
  - args and kwargs 
    <pre>
     # 파이썬의 기본 문법이다.
     def plus(a, b, *args, **kwargs):
        print(args)
        print(kwargs)
        return a + b
     # plus(1,2,3,4,5,1,2,3, hello = True, hello2 = False)와 같은 함수의 사용이 가능하다
     
     def print_args(*args):
        result = 0
        for number in args:
            result += number
        return result
        
     print(print_args(1,2,3,4,5,6,7,8,9,10)) 
     # sum 1 ~ 10
    </pre>
    
   - 객체 지향
     > 클래스 안에 선언핸 함수를 메서드라고 칭하며 모든 메서드들은
       하나의 argument를 기본적으로 함께 사용한다 => self를 사용한다
       이때 클래스를 통해 생성된 인스턴스 그 자신을 나타낸다. 자바의 this와 같다.
        <pre>
        class Car():
            wheels = 4
            doors = 4
            windows = 4
            seats = 4
            
            def start(self):
                print(self)
                # print(self.color)
                print("I started")
        
        avante = Car()
        avante.color = "Silver"
        avante.start()
        
        # print(avante)를 통해 나오는 결과는 객체를 생성했을때 기본적으로 생성되는 내장함수중 하나를 호출한다  => __str__
        # print(dir(avante))를 통해 기본 생성되는 객체의 함수를 확인 가능하다.
        </pre>
        <pre>
        class Car():
            # 기본 함수인 __init__ 함수(메서드)를 재정의(over write) 하였다.
            def __init__(self, *args, **kwargs): 
                self.wheels = 4
                self.doors = 4
                self.windows = 4
                self.seats = 4
                self.color = kwargs.get("color", "black") # keword arguments중 "color"를 찾고 없으면 기본값으로 black을 할당한다.
                self.price = kwargs.get("price", "$999)
            
            def start(self):
                print(self)
                print(self.color)
                print("I started")
            
            def __str__(self):
                return f"over write and Car with {self.wheels} wheels"
            
        kona = Car(color = "Red", price = "$500")
        kona.color = "Green"
        
        # kona.start()
        # print(dir(Car))
        
        # print(kona)
        # print(kona.color, kona.price)
        
        mini = Car()
        # print(mini.color, mini.price)
        
        ######### inheritance or extends ###########
        class Convertible(Car):
            def __init(self, **kwargs):
                super().__init__(**kwargs) # 자바에서 상위 클래스에 접근하는 super와 동일하다
                self.time = kwargs.get("time", 10)
            
            def take_off(self):
                return "taking off the roof"
            
            def __str__(self):
                return "car with no roof"
        
        porche = Convertible(color="Yellow", price ="$5000")
        
        print(porche.take_off())
        print(porche.wheels)
        print(porche)
        print(porche.color)
        print(porche.time)                 
        </pre>

