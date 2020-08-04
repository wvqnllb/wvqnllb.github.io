# Java Reflection


学习了基本了Java反射特性。

<!--more-->

## Intro
The reflection library gives you a very rich and elaborate toolset to write programs that manipulate Java code dynamically. It is of interest mainly to tool builders.

A program that can analyze the capabilities of classes is called **reflective**.

Usage:
* Ananlyze the capabilities of classes at runtime
* Inspect objects at runtime
* Implement generic array manipulation code
* Take advantage of Method object that work just like funcion pointers in languages such as C++


## The *Class* Class
By working with the special Java class, you can access run time type identification information on all objects.

The following code shows different way of get a Class Object
```java
Employee e;
. . .
Class c1 = e.getClass();
```
```java
Class c2 = T.class;
//T is any Java type;
//e.g. java.util.Random
```
```java
Class c3 = Class.forName(className);
//className 
//e.g. "java.util.Random"
//Any method that calls this method needs a throws declearation.
```
Comprehensive example
```java
Employee e = new Employee();
Class c1 = e.getClass();
String Name = c1.getName();
Class c2 = Employee.class();
Class c3 = Class.getClass(Name);
// c1 == c2 == c3
Constructor constructor = c1.getConstructor();

Object obj = constructor.newInstance(Object...Params);
```

## Using Reflection to Analyze the Capabilities of Classes
Reflection mechanism allows you to examine the structure of a class
* Field
* Method
* Constructor

Using the following program, we can analyze any class that the Java interpreter can load.
```java
package reflection;



import java.util.*;
import java.lang.reflect.*;



public class ReflectionTest {
    public static void main(String[] args) throws ReflectiveOperationException{
        String name;
        if (args.length > 0) {
            name = args[0];
        } else {
            var in = new Scanner(System.in);
            System.out.println("Enter class name (e.g. java.util.Date)");
            name = in.next();
        }

        Class c1 = Class.forName(name);
        Class superc1 = c1.getSuperclass();

        String modifiers = Modifier.toString(c1.getModifiers());
        // 打印出类前的modifier
        if (modifiers.length() > 0) {
            System.out.print(modifiers + " ");
        }
        // 打印类名 和 父类名
        System.out.print("class " + name + " ");
        if (superc1 != null && superc1 != Object.class) {
            System.out.print("extends " + superc1.getName());
        }

        System.out.print("\n{\n");
        printConstructors(c1);
        System.out.println();
        printMethods(c1);
        System.out.println();
        printFields(c1);
        System.out.println("}");

    }
    // 打印Constructor的 Modifier + name + parameterTpyes
    public static void printConstructors(Class c1) {
        Constructor[] constructors = c1.getDeclaredConstructors();
        for (Constructor c : constructors) {
            String name = c.getName();
            System.out.print("  ");
            String modifiers = Modifier.toString(c.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.print(name + "(");

            Class[] paramTypes  = c.getParameterTypes();
            for (int i = 0; i < paramTypes.length; i++) {
                if (i > 0) {
                    System.out.print(",");
                }
                System.out.print(paramTypes[i].getName());
            }
            System.out.println(");");
        }
    }
    // 打印Method的 modifier + returnTypes + name + parameterTypes
    private static void printMethods(Class c1) {
        Method[] methods = c1.getMethods();
        for (Method m : methods) {
            Class retType = m.getReturnType();
            String name = m.getName();

            System.out.print("  ");


            String modifiers = Modifier.toString(m.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }

            System.out.print(retType.getName() + " " + name + "(");

            Class[] paramTypes = m.getParameterTypes();

            for (int j = 0; j < paramTypes.length; j++) {
                if (j > 0) {
                    System.out.print(", ");
                }
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }

    // 打印Fields 的 Modifier parameterTypes Name
    private static void printFields(Class c1) {
        Field[] filelds = c1.getFields();
        for (Field f : filelds) {
            Class type = f.getType();
            String name = f.getName();
            System.out.print("  ");
            String modifiers = Modifier.toString(f.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.println(type.getName() + " " + name + ";");
        }

    }
}

```

## Using Reflection to Analyze Object at Runtime
Of course, it is easy to look at the contents of a specific field of an object whose name and type are known when writing a program. But reflection lets you look at fields of objects that were not known at compile time.
```java
var harry = new Employee ("Harry Hacker", 50000, 10, 1, 1989);
var jennie = new Employee("Jennie Kim", 5000, 16, 1, 1996);
Class c1 = harry.getClass();
Field f = c1.getDeclaredField("name");
Object v= f.get(harry);
f.set(harry, "Harry Potter");
//Only possible when "name" field is not private;
f.setAccessible(true);
//Now ok if call f.get(harry);
```

Here is an example of a universal toString Method
(是一个非常好的加深理解的例子)

ObjectAnalyzerTest.java

```java
package objectAnalyzer;
import java.util.*;

/**
 * This program uses reflection to spy on object.
 */

public class ObjectAnalyzerTest {
    public static void main(String[] args) throws ReflectiveOperationException{
        var squares = new ArrayList<Integer>();
        System.out.println(Integer.class.isPrimitive());
        for (int i = 1; i <= 5; i++) {
            squares.add(i * i);
        }
        System.out.println(new ObjectAnalyzer().toString(squares));
    }
}
```
ObjectAnalyzer.java
```java
package objectAnalyzer;

import java.lang.reflect.AccessibleObject;
import java.lang.reflect.Array;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.ArrayList;

public class ObjectAnalyzer {
    private ArrayList<Object> visited = new ArrayList<>();

    public String toString(Object obj) throws ReflectiveOperationException{
        if (obj == null) return "null";
        if (visited.contains(obj)) {
            return "...";
        }
        visited.add(obj);

        Class c1 = obj.getClass();
        if (c1 == String.class) {
            return (String) obj;
        }
        if (c1.isArray()) {
            String r = c1.getComponentType() + "[]{";
            for (int i = 0; i < Array.getLength(obj); i++) {
                if (i > 0) {
                    r += ",";
                }
                Object val = Array.get(obj, i);
                if (c1.getComponentType().isPrimitive()) r += val;
                else r += toString(val);
            }
            return r + "}";
        }

        String r = c1.getName();
        do {
            r += "[";
            Field[] fields = c1.getDeclaredFields();
            AccessibleObject.setAccessible(fields, true);
            for (Field f : fields) {
                if (!Modifier.isStatic(f.getModifiers())) {
                    if (!r.endsWith("[")) r+= ",";
                    r += f.getName() + "=";
                    Class t = f.getType();
                    Object val = f.get(obj);
                    if (t.isPrimitive()) r += val;
                    else r+= toString(val);
                }
            }
            r += "]";
            c1 = c1.getSuperclass();

        } while (c1 != null);

        return r;
    }
}
```

## Using Reflection to Write Generic Array Code
Allow you to create arrays dynamically.

Problem: How can we write such a generic method?
```java
var a = new Employee[100];
a = Arrays.copyOf(a, 2 * a.length);
```
One way is to create a new Object[] array in the method and return it. However, an array of objects cannot be cast to an array of employees. The virtual machine would generate a ClassCastException at runtime. The point is that, as we mentioned earlier, a Java array remembers the type of its entries -- that is the element type used in the new expression that created it. It is legal to cast an Employee[] temporarily to an Object[] array and then cast it back. But an array that started its lef as an Object[] array can never be cast into an Employee[] array.

Here is a good way to implement copy using reflection.
```java
public static Object goodCopyOf(Object a, int newLength) {
    Class c1 = a.getClass();
    if (!c1.isArray()) return null;
    Class componentType = c1.getComponentType();
    int length = Array.getLength(a);
    Object newArray = Array.newInstance(componentType, newLength);
    System.arraycopy(a,0,newArray,0,Math.min(length, newLength));
}
```

## Invoking Arbitrary Methods and Constructors

Recall that we can inspect a field of an object with getMethod of the Field class. Similarly, the Method class has an invoke method that lets you call the method that is wrapped in the current Method object.
```java
Object invoke(Object obj, Object... args)
```
If the return type is a primitive type, the invoke method will return the wrapper type instead.
Two ways to get the target Method object
```java
Class c = a.getClass();
Method[] methods = c.getDeclaredMethods();
// Then choose the specific method;
// or
Method method = c.getMethod(String name, Class... parameterTypes);
```



