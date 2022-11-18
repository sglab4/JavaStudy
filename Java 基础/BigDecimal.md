在 BigDecimal 中，**通过一个”无标度值” intval 和一个”标度” scale 来表示一个浮点数。**

![image-20211126160218408](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202111261602459.png)

scale 的解释：如果scale为零或正值，则该值表示这个数字小数点右侧的位数。如果scale为负数，则该数字的真实值需要乘以10的该负数的绝对值的幂。例如，scale为-3，则这个数需要乘1000，即在末尾有3个0。

如123.123，那么如果使用BigDecimal表示，那么他的无标度值为123123，他的标度为3。对于 0.1，使用无标度 1 和标度 1 进行表示。

接下来讨论为什么使用 new BigDecimal(double) 会出现精度丢失问题，因为**当我们使用new BigDecimal(0.1)创建一个BigDecimal 的时候，其实创建出来的值并不是正好等于0.1的。**而是0.1000000000000000055511151231257827021181583404541015625。这是因为doule自身表示的只是一个近似值。

应该使用 new BigDecimal(String)，原因是使用 String 进行创建后，会将 String 转换为 char 数组，然后再将该字符数组表示的浮点数转换为使用无标度 intVal 和标度 scale 进行表示。