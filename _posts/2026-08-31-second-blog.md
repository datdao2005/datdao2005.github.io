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

```c

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

Nôm na là cái hàm này sẽ chèn cái header "SDRM" vào các file bị mã hóa, nhưng mà cơ chế mã hóa ra sao thì lại chưa nằm ở chỗ này.
Tiếp tục mò mấy folder khác, có folder "node" chứa 3 file Javascript, trong đó file cuối cùng khá sú:

```js
import {
    t as x, b as g
}

from"../chunks/DIzuPXZE.js";
import {
    i as X
}

from"../chunks/Bmu4QhlG.js";
import {
    h as T, a2 as Y, a3 as J, a4 as Z, a5 as ee, C as te, a6 as ne, w as ae, a7 as re, a8 as se, I as u, z as ie, Q as _, a9 as I, aa as O, O as p, P as w, ab as oe
}

from"../chunks/fua1tWYb.js";
import {
    b as le, l as ce, e as N
}

from"../chunks/BC_VcuN-.js";
import {
    i as S
}

from"../chunks/XEl44-Fd.js";
const fe = Symbol("is custom element"), ue = Symbol("is html");
function de(e) {
    if (T)  {
        var t = !1, n = () => {
            if (!t)  {
                if (t = !0, e.hasAttribute("value"))  {
                    var r = e.value;
                    D(e, "value", null), e.value = r
                }

                if (e.hasAttribute("checked"))  {
                    var s = e.checked;
                    D(e, "checked", null), e.checked = s
                }

            }

        };
        e.__on_r = n, Y(n), le()
    }

}

function D(e, t, n, r) {
    var s = be(e);
    T&&(s[t] = e.getAttribute(t), t === "src"||t === "srcset"||t === "href"&&e.nodeName === "LINK")||s[t] !== (s[t] = n)&&(t === "loading"&&(e[J] = n), e.removeAttribute(t))
}

function be(e) {
    return e.__attributes??(e.__attributes = {
        [fe]:e.nodeName.includes("-"), [ue]:e.namespaceURI === Z
    }

    )
}

function ye(e, t, n = t) {
    var r = ee();
    ce(e, "input", s => {
        var o = s?e.defaultValue:e.value;
        if (o = L(e)?R(o):o, n(o), r&&o !== (o = t()))  {
            var d = e.selectionStart, y = e.selectionEnd;
            e.value = o??"", y !== null&&(e.selectionStart = d, e.selectionEnd = Math.min(y, e.value.length))
        }

    }

    ), (T&&e.defaultValue !== e.value||te(t) == null&&e.value)&&n(L(e)?R(e.value):e.value), ne(() => {
        var s = t();
        L(e)&&s === R(e.value)||e.type === "date"&&!s&&!e.value||s !== e.value&&(e.value = s??"")
    }

    )
}

function L(e) {
    var t = e.type;
    return t === "number"||t === "range"
}

function R(e) {
    return e === ""?null: + e
}

async function h(e, t = {}, n) {
    return window.__TAURI_INTERNALS__.invoke(e, t, n)
}

async function _e(e = {}

) {
    return typeof e == "object"&&Object.freeze(e), await h("plugin:dialog|open", {
        options:e
    }

    )
}

let c, v = null;
function P() {
    return(v === null||v.byteLength === 0)&&(v = new Uint8Array(c.memory.buffer)), v
}

let M = 0;
function C(e, t) {
    const n = t(e.length * 1, 1) >>> 0;
    return P().set(e, n/1),M=e.length,n}function ve(e,t){return e=e>>>0,P().subarray(e/1, e / 1 + t)
}

function me(e, t) {
    const n = C(e, c.__wbindgen_malloc), r = M, s = C(t, c.__wbindgen_malloc), o = M, d = c.inject_key(n, r, s, o);
    var y = ve(d[0], d[1]).slice();
    return c.__wbindgen_free(d[0], d[1] * 1, 1), y
}

async function ge(e, t) {
    if (typeof Response == "function"&&e instanceof Response)  {
        if (typeof WebAssembly.instantiateStreaming == "function") try {
            return await WebAssembly.instantiateStreaming(e, t)
        } catch(r) {
            if (e.headers.get("Content-Type") != "application/wasm")console.warn("`WebAssembly.instantiateStreaming` failed because your server does not serve Wasm with `application/wasm` MIME type. Falling back to `WebAssembly.instantiate` which is slower. Original error:\n", r);
            else throw r
        }

        const n = await e.arrayBuffer();
        return await WebAssembly.instantiate(n, t)
    } else {
        const n = await WebAssembly.instantiate(e, t);
        return n instanceof WebAssembly.Instance? {
            instance:n, module:e
        }

        :n
    }

}

function pe() {
    const e = {};
    return e.wbg = {}, e.wbg.__wbindgen_init_externref_table = function() {
        const t = c.__wbindgen_export_0, n = t.grow(4);
        t.set(0, void 0), t.set(n + 0, void 0), t.set(n + 1, null), t.set(n + 2, !0), t.set(n + 3, !1)
    }, e
}

function we(e, t) {
    return c = e.exports, H.__wbindgen_wasm_module = t, v = null, c.__wbindgen_start(), c
}

async function H(e) {
    if (c !== void 0) return c;
    typeof e < "u"&&(Object.getPrototypeOf(e) === Object.prototype? {
        module_or_path:e
    } = e:console.warn("using deprecated parameters for the initialization function; pass a single object instead")), typeof e > "u"&&(e = new URL("" + new URL("../assets/key_injector_bg.X6nK8EHb.wasm", import.meta.url).href, import.meta.url));
    const t = pe();
    (typeof e == "string"||typeof Request == "function"&&e instanceof Request||typeof URL == "function"&&e instanceof URL)&&(e = fetch(e));
    const {
        instance:n, module:r
    } = await ge(await e, t);
    return we(n, r)
}

var he = x('<label for="email" class="text-lg font-medium text-gray-700">Enter your email to continue</label>'), 
xe = x('<p class="text-sm text-red-500">Please enter a valid email address</p>'), 
Ae = x('<div class="my-auto flex justify-evenly"><button class="cursor-pointer rounded-lg border bg-white px-6 py-4 text-lg shadow-md transition hover:scale-105 hover:shadow-lg">Encrypt</button> <button class="flex cursor-not-allowed flex-col rounded-lg border bg-gray-100 px-6 py-4 text-lg shadow-md">Decrypt</button></div>'), 
ke = x('<main class="container mx-auto"><div class="flex min-h-screen flex-col border-gray-900 p-8 text-center text-gray-900"><div class="flex flex-col gap-2"><h1 class="text-3xl font-bold tracking-wide">Simple DRM</h1> <p class="text-xl italic"><span class="relative inline-block before:absolute before:-inset-1 before:block before:-skew-y-6 before:bg-pink-500"><span class="relative font-semibold text-white">free</span></span><span class="pl-2 tracking-wide">Edition</span></p></div> <div class="my-8 flex flex-col items-center gap-4"><!> <input id="email" type="email" placeholder="your-email@example.com" class="w-full max-w-md rounded-lg border border-gray-300 px-4 py-3 text-center text-lg shadow-sm transition focus:border-pink-500 focus:ring-2 focus:ring-pink-200 focus:outline-none"> <!></div> <!> <footer class="absoulte right-0 botton-0 left-0 pb-4 text-center"><p class="text-sm text-gray-500">Powered by Tauri</p></footer></div></main>');
function Te(e, t) {
    ae(t, !1);
    const n = O();
    let r = O("");
    (async() => await H())();
    async function s(a) {
        const b = new TextEncoder().encode(a.trim().toLowerCase()), 
        l = await crypto.subtle.digest("SHA-256", b);
        return new Uint8Array(l)
    }

    function o(a, i) {
        const b = new Uint8Array(a.length);
        for(let l = 0; l < a.length;l++)
            b[l] = a[l]^i[l%i.length];
        return b
    }

    async function d() {
        if (!u(n)) return;
        const a = await _e( {
            multiple:!0
        }

        );
        if (Array.isArray(a)&&a.length > 0)  {
            const i = a.map(async m => {
                const $ = await h("read_file", {
                    path:m
                }

                ).then(f => f).catch(f => {
                    throw console.error(f), f
                }

                ), j = await s(u(r)), G = o($, j), Q = me(j, G);
                return await h("save_enc_file", {
                    path:m, contents:Q
                }

                ).then(f => f).catch(f => {
                    throw console.error(f), f
                }

                )
            }

            ), l = (await Promise.all(i)).filter(m => m !== void 0).length, K = await h("get_app_data_dir");
            alert(`$ {l} files are encrypted. Check the '${K}'`)
        }

    }

    function y() {
        alert("Decryption is only available in the Pro edition.")
    }

    re(() => u(r), () => {
        I(n, /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(u(r).trim()))
    }

    ), se(), X();
    var A = ke(), U = p(A), k = _(p(U), 2), W = p(k); {
        var z = a => {
            var i = he();
            g(a, i)
        };
        S(W, a => {
            u(n)||a(z)
        }

        )
    }

    var E = _(W, 2);
    de(E);
    var V = _(E, 2); {
        var q = a => {
            var i = xe();
            g(a, i)
        };
        S(V, a => {
            u(r).trim().length > 0&&!u(r).includes("@")&&a(q)
        }

        )
    }

    w(k);
    var B = _(k, 2); {
        var F = a => {
            var i = Ae(), b = p(i), l = _(b, 2);
            w(i), N("click", b, d), N("click", l, y), g(a, i)
        };
        S(B, a => {
            u(n)&&a(F)
        }

        )
    }

    oe(2), w(U), w(A), ye(E, () => u(r), a => I(r, a)), g(e, A), ie()
}

export {
    Te as component
};
```

Tới đây thì mọi thứ có tương lai hơn rồi.
Challenge mình sẽ để link bên dưới để các bạn trải nghiệm thử nha, cảm ơn mọi người đã đọc cái WU xàm xí này của mình, chúc mọi người vui vẻ và tránh bị vấp cỏ như mình ngày trước ! :)))



