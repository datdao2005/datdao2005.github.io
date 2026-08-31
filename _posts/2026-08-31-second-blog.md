---
title: "Hacktheon Sejong Final 2025"
date: 2026-08-31 00:23:00 +0700
categories: [rev, ctf]
tags: [rev]
---

# Introduction
Well, chào mọi người, mình là Đạt, một sinh viên năm cuối cùng học rev, cái blog này thì mình mới tạo chắc được hơn 1 tiếng, ừ chắc vậy.

Nay cũng không có gì, kiểu viết cái post để trả nợ những lần bị mình delay quá lâu, cụ thể là 3 năm. Như tiêu đề, đây là tổng hợp những bài mình đã từng làm (và chắc chắn là chưa solve được) trong đợt tham dự vòng chung kết Hacktheon Sejong năm 2025 tại Hàn Quốc.

# Simple DRM
Bài này không nhớ mô tả là gì nên thôi vô vấn đề chính nha :V
Bài cho chúng ta 1 file zip chứa 1 file PE simple_drm_v0.1.0_x64_setup.exe để setup chương trình và 4 file .png.enc, khả năng là 4 file ảnh bị mã hóa.
![Image Alt Text](/assets/img/posts/drm.png)

Sau khi mình chạy file thực thi kia xong thì có hiện ra một app tên simple-drm.exe, app này yêu cầu mình nhập một email, không nhất thiết là email real, xong rồi cho mình chọn encrypt và decrypt (tất nhiên là chỉ có mỗi encrypt tại vì nút decrypt bị "hư" rồi :)))) 

Rồi xong, bắt đầu vô công cuộc rev xem coi tại làm sao, mình mở IDA lẫn Ghidra lên xem và rốt cuộc là... không có gì hết. Mình quăng app lên Pháo hoa cam (chắc bạn cũng biết mình nhắc tới cái gì rồi) để hỏi nó xem chỗ nào nhập input, thì có một chi tiết mà mình bây giờ lẫn hồi đi thi quên để ý là app này là Tauri app.

Vậy Tauri là cái gì ?
Theo trang Tauri thì đây là một framework xây dựng ứng dụng nhỏ dùng cho đa nền tảng, dùng backend chủ yếu là Rust và frontend là bộ ba khá quen thuộc: HTML + CSS + JS.

Okay, vậy là cũng biết sơ sơ thông tin rồi, vậy làm sao dịch ngược ứng dụng này khi mà bỏ vô các trình dịch ngược đều quăng ra một mớ source mà không thấy chỗ biến đổi input, logic các thứ ? Để giải đáp được thì:

![Image Alt Text](/assets/img/posts/images.jpg)

Well, mình search "Tauri app reverse engineering" thì tìm ra cái trang này: https://lib.rs/crates/tauri-dumper
Đại khái là tool này sẽ giúp cho mình dump ra tất cả các thứ được nén trong ứng dụng Tauri, và bài này khả năng là vận dụng kỹ năng sách giáo khoa cho tool đây.

Sau khi chạy tool xong, mình nhận được kết quả là một folder tên assets, bên trong đó có gì thì các bạn coi hình bên dưới nha: 

![Image Alt Text](/assets/img/posts/res1.png)

Tiếp tục đi sâu vào thư mục _app thì vào tiếp thư mục tên immutable, trong đây chứa thêm 4 folders nữa, nhiều vl:
![Image Alt Text](/assets/img/posts/res2.png)

Lục tìm một hồi thì cái đập vào mắt mình là một file wasm aka assembly của web:
![Image Alt Text](/assets/img/posts/res3.png)

Bỏ vào Ghidra thì thấy có 1 hàm có tiềm năng:

``` C

int export::inject_key(undefined4 param1,uint param2,undefined4 param3,uint param4)

{
  uint uVar1;
  undefined4 param1_00;
  int param1_01;
  uint uVar2;
  uint local_1c;
  int iStack_18;
  uint local_14;
  
  uVar1 = 0;
  param1_00 = 0;
  uVar2 = param2 + param4 + 0xc;
  if ((int)uVar2 < 0) {
code_r0x80002b1b:
    unnamed_function_26(param1_00,uVar2,&PTR_s_src/lib.rs_ram_00100060_ram_0010006c);
    do {
      halt_trap();
    } while( true );
  }
  if (uVar2 == 0) {
    local_1c = 0;
    iStack_18 = 1;
  }
  else {
    param1_00 = 1;
    iStack_18 = unnamed_function_31(uVar2,1);
    if (iStack_18 == 0) goto code_r0x80002b1b;
    local_1c = uVar2;
    if (3 < uVar2) goto code_r0x80002886;
  }
  local_14 = 0;
  unnamed_function_11(&local_1c,0,4);
  uVar1 = local_14;
code_r0x80002886:
  *(undefined4 *)(uVar1 + iStack_18) = 0x4d524453;
  uVar2 = uVar1 + 4;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_0010007c);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param2 >> 0x18);
  uVar2 = uVar1 + 5;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_0010008c);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param2 >> 0x10);
  uVar2 = uVar1 + 6;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_0010009c);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param2 >> 8);
  uVar2 = uVar1 + 7;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_001000ac);
  }
  *(char *)(iStack_18 + uVar2) = (char)param2;
  uVar2 = uVar1 + 8;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_001000bc);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param4 >> 0x18);
  uVar2 = uVar1 + 9;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_001000cc);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param4 >> 0x10);
  uVar2 = uVar1 + 10;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_001000dc);
  }
  *(char *)(iStack_18 + uVar2) = (char)(param4 >> 8);
  uVar2 = uVar1 + 0xb;
  if (local_1c == uVar2) {
    local_14 = uVar2;
    unnamed_function_13(&local_1c,&PTR_s_src/lib.rs_ram_00100060_ram_001000ec);
  }
  *(char *)(iStack_18 + uVar2) = (char)param4;
  local_14 = uVar1 + 0xc;
  if (local_1c - local_14 < param2) {
    unnamed_function_11(&local_1c,local_14,param2);
  }
  if (param2 != 0) {
    memory_copy(0,0,param2,param1,iStack_18 + local_14);
  }
  local_14 = param2 + local_14;
  if (local_1c - local_14 < param4) {
    unnamed_function_11(&local_1c,local_14,param4);
  }
  uVar2 = local_14;
  param1_01 = iStack_18;
  uVar1 = local_1c;
  if (param4 != 0) {
    memory_copy(0,0,param4,param3,iStack_18 + local_14);
    unnamed_function_36(param3,param4);
  }
  uVar2 = param4 + uVar2;
  if (param2 != 0) {
    unnamed_function_36(param1,param2);
  }
  if (uVar2 < uVar1) {
    if (uVar2 == 0) {
      unnamed_function_36(param1_01,uVar1);
      param1_01 = 1;
    }
    else {
      param1_01 = unnamed_function_28(param1_01,uVar1,uVar2);
      if (param1_01 == 0) {
        unnamed_function_26(1,uVar2,&
                                    PTR_s_/usr/local/cargo/registry/src/in_ram_001000fc_ram_00100168
                           );
        do {
          halt_trap();
        } while( true );
      }
    }
  }
  return param1_01;
}

```
{: .scroll}


Challenge mình sẽ để link bên dưới để các bạn trải nghiệm thử nha, cảm ơn mọi người đã đọc cái WU xàm xí này của mình, chúc mọi người vui vẻ và tránh bị vấp cỏ như mình ngày trước ! :)))



