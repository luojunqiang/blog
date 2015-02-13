### Decimal (boost::multiprecision::cpp_dec_float20)

    2^60 = 1152921504606846976   // 可以支持18位有效数字
    2^4 = 16
    
    MAX(float20.backend().order(),0)+1  // 可以获得float20的整数位数，设为 A。
    B = 18 - A;  // 可以算出还可以支持的小数位数
    C = MIN(B, 2^4);  // 可以得出需要的小数位数，因为最多支持16位小数
    
    float20 * 10^C 的结果转换为int64 放在前60位中
    后4位中存放 C 的值
    
    另外对一些非法的float值需要参考boost中的代码做一些检查
    boost内部一些实现细节可以参考一下下面测试代码的输出：
    http://coliru.stacked-crooked.com/a/bb8505899d2040d5

