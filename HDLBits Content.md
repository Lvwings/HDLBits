# HDLBits Content

## Verilog language

This part review the basic knowledge of Verilog language.

### Vectors

> Vectors are used to group related signals using one name to make it more convenient to manipulate. 

<img src="C:\Users\Lwing\AppData\Roaming\Typora\typora-user-images\image-20200214105252957.png" alt="image-20200214105252957" style="zoom:50%;" />

The assignment of the separate 1-bit output can also be coded as

```verilog
assign {o2, o1, o0} = vec;
```

#### Declaring Vectors

> In Verilog, once a vector is declared with a particular endianness 【字节顺序】, it must always be used the same way. e.g.

```verilog
wire [3:0] vec; //  big vector【大端模式】 
assign vec[0:3] = xx; // is illegal
```

The part-select operator can be used to access a portion of a vector:

```verilog
reg [255:0] data = 0;
genvar i;
generate	
    for (i = 0; i < 16; i = i+1)
        data[i*16 +: 16] = i;	// data[15:0],data[31:16] ...
endgenerate
```

#### Implicit nets

> In Verilog, net-type signals can be implicitly created by an `assign` statement or by attaching something undeclared to a module port. 

```verilog
wire [2:0] a, c;    // Two vectors
assign a = 3'b101;  // a = 101
assign b = a;       // b = 1 implicitly-created wire
assign c = b;       // c = 001  <-- bug
```

> Disabling creation of implicit nets can be done using the ``default_nettype none` directive，which codes on the top of the file. 

Adding ``default_nettype none` would make the assignment of b an error, which makes the bug more visible. 

#### Memory  type

This type is also called as "packed arrays" and **can not be synthesize**. The declaration is

```verilog
parameter wordsize = 16, memsize = 256;
// 256 elements , each of which is a 16-bit reg.
reg [wordsize-1:0] mem [memsize-1:0]; 
```

> Verilog 通过reg建立数组来对存储器建模，可以描述RAM，ROM和reg文件。上述声明相当于构建了名为mem的存储器，其中包含256个16位寄存器。

```verilog
 mem = 0； // is illegal 
```

System tack`$readmemb` and `$readmemh` are used to initialize the memory type.

```verilog
initial begin
// initialize mem[start] to mem[end] according to file value
$readmemh("file_name", mem, start, end);
end
```

#### Concatenation operator

> The replication operator allows repeating a vector and concatenating them together: `{num{vector}}`

```verilog
assign out = { {24{in[7]}}, in };
```

------

### Modules

Module is a black box, only the ports on the module are important. You do not need to know the code inside the module. There are two commonly-used methods to instantiate 【例化】modules（ connect a wire to a port）: by position or by name.

```verilog
mod_a instance1 ( wa, wb, wc ); // By position
mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) ); 	// By name
```

#### Generate statement

`Generate for-loop` will generate an instance for each iteration.  we can use this feather in some situations which needs to  repeat instance . A example of code that requires generate for is:

```verilog
module add16 #(parameter bitwidth = 16)
    ( input [bitwidth - 1:0] a, input [bitwidth - 1:0] b, input cin, output [bitwidth - 1:0] sum, output cout );
    wire [bitwidth:0] fcout;	// store the cout of each iteration 
    genvar i;
    generate 
        for (i = 0; i < bitwidth; i = i + 1)
            begin : add 		// name is necessary
                 add1 add (a[i],b[i],fcout[i],sum[i],fcout[i+1]);             
            end
    endgenerate
	assign fcout[0] = 0;
    assign cout = fcout[bitwidth];
endmodule
// A 1-bit full adder module
module add1 ( input a, input b, input cin, output sum, output cout );
   wire [1:0] fsum;
   assign fsum = a + b + cin;
   assign sum = fsum[0];
   assign cout = fsum[1];
endmodule
```

`Generate` helps us to reduce the repetition code. 