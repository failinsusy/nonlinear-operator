## 将指数函数的基数由e化为2
该方法将旨在消除指数计算过程中对内存的需求，因此，我们不使用泰勒级数来计算指数值，因为泰勒级数对内存的需求很高，而且计算复杂度也很高。因此，我们简化了指数函数的数学计算，以消除硬件设计中的内存使用。

将$e^x$转换为$2^{x{\log_2e}}$,由上式可知，$\log_2e$可以表示为一个常数，因为它表示欧拉数的简单对数，值。这就消除了在硬件上执行$\log_2$操作的需要。

将x与$\log_2e$的乘积分成整数y和小数z两部分，则相当于求:$e^x=2^{y+z}=2^y*2^z$

指数函数被简化为两个数学运算，即以2为底计算整数部分和分数部分的幂，以及一次乘法运算。左移运算可以在整数部分的硬件设计中有效地执行。现在，以2为底的分数幂的计算是唯一的复杂计算部分。

在这项工作中，分数部分z被限制在﹣0.5到＋0.5的范围内。由[文献](https://ieeexplore.ieee.org/document/9937238)数据和结论可知，
多项式为：$2^z=a+bx+cx^2+bx^3$
其中a=0.99992807,b=0.69326098, c=0.24261112,d=0.00517166

实现导致［-6,15］范围内的值的最大相对误差为$10^{-3}$，最大绝对误差为$6.42*10^{-5}$。

## Verilog实现如下
```
module exponent(
        input wire clk,
        input wire rst_n,
        input wire[31:0] x,
        output reg[31:0] data);
        
        parameter y=32'h0001_7154,
                   a=32'h0000_fffb,
                   b=32'h0000_b179,
                   c=32'h0000_3e1b,
                   d=32'h0000_0152;
                   
       reg[31:0] term_1;//ln2的倒数与x的乘积
       reg[31:0] term_2;//小数部分对应的2的指数的值
       reg[31:0] x2;    //小数部分
       reg[31:0] x_sqaure;
       reg[31:0] x_cube;
       reg[63:0] temp;  //临时结果
       
always @(posedge clk or negedge rst_n)
       begin
       if(!rst_n)
            begin
                term_1<=32'h0;
                term_2<=32'h0;
                x2<=32'h0;
                data<=32'h0;
            end
        else
           begin
            temp=x*y;
            term_1=temp>>16;
            x2={16'h0000,term_1[15:0]};
            temp=(x2*x2);
            x_sqaure=temp>>16;
            temp=x_sqaure*x2;
            x_cube  =temp>>16;
            term_2=a+((b*x2)>>16)+((c*x_sqaure)>>16)+((d*x_cube)>>16);
            data=term_2<<term_1[31:16]; 
          end
        end
endmodule
```