# JAVA JVM助记符

ldc：将int/float/String类型的常量值从常量池中推送至栈顶（栈顶的值是即将要用的）

bipush：将单字节（-128 ~ 127）的常量值从常量池中推至栈顶

sipush：将一个短整型（-32768 ~ 32767）的常量值从常量池中推至栈顶

iconst_1：将int型的常量值1从常量池中推至栈顶（jvm专门为0/1/2/3/4/5这5个数字开的助记符），iconst_m1则表示的是-1

anewarray：创建一个引用类型（如类、接口、数组）的数组，并将其引用值推至栈顶

newarray：创建一个指定的原始类型（如int/float）的数组，并将其引用值推至栈顶

getstatic（读取类的静态字段）

putstatic（设置类的静态字段）

invokestatic（调用静态方法）