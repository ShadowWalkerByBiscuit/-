1-常量命名全部大写并用_分隔每个单词。  
2-抽象类要以abstract开头或者base结尾，异常类要以Exception结尾，测试类要以Test结尾。  
3-接口类中的接口定义，前面不用加任何修饰符号，连public都不需要。  
4-枚举类后面要加上Enum的后缀。  
5-在long或者Long型定义的时候，后面的数字要用L而不能用i，因为会混淆成1.  
6-为了避免精度丢失的风险，进制用bigdecimal转换double类型数据，正确的方法是把double转成String之后再转成Bigdecimal。  
7-在类中加上toString函数的时候，如果是继承了其他类，记得加上super.toString方法，方便排查。
