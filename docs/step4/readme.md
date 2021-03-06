### ๐ฏ ํ์ต ๋ชฉํ

![image](../image/step4/image01.png)

- AWS ์์์ ๋คํธ์ํฌ๋ฅผ ๊ตฌ์ฑํ๋ฉฐ, ๋คํธ์ํฌ ๊ธฐ๋ณธ ๊ฐ๋๋ค์ ํ์ตํด๋ด๋๋ค.
- ์ปจํ์ด๋๋ฅผ ํ์ตํ๊ณ  3 tier๋ก ์ด์ํ๊ฒฝ์ ๊ตฌ์ฑํด๋ด๋๋ค.
- ๊ฐ๋ฐ ํ๊ฒฝ์ ๊ตฌ์ฑํด๋ณด๊ณ  ์ง์์  ํตํฉ์ ๊ฒฝํํด๋ด๋๋ค.

---

### 1. ์๋น์ค ๊ตฌ์ฑํ๊ธฐ

โ๏ธ ์๊ตฌ์ฌํญ
- ์น ์๋น์ค๋ฅผ ์ด์ํ  ๋คํธ์ํฌ ๋ง ๊ตฌ์ฑํ๊ธฐ
- ์น ์ ํ๋ฆฌ์ผ์ด์ ๋ฐฐํฌํ๊ธฐ

๐ ๋ง ๊ตฌ์ฑ
- VPC ์์ฑ
    - CIDR์ C class(x.x.x.x/24)๋ก ์์ฑ. ์ด ๋, ๋ค๋ฅธ ์ฌ๋๊ณผ ๊ฒน์น์ง ์๊ฒ ์์ฑ
- Subnet ์์ฑ
    - ์ธ๋ถ๋ง์ผ๋ก ์ฌ์ฉํ  Subnet : 64๊ฐ์ฉ 2๊ฐ (AZ๋ฅผ ๋ค๋ฅด๊ฒ ๊ตฌ์ฑ)
    - ๋ด๋ถ๋ง์ผ๋ก ์ฌ์ฉํ  Subnet : 32๊ฐ์ฉ 1๊ฐ
    - ๊ด๋ฆฌ์ฉ์ผ๋ก ์ฌ์ฉํ  Subnet : 32๊ฐ์ฉ 1๊ฐ
- Internet Gateway ์ฐ๊ฒฐ
- Route Table ์์ฑ
- Security Group ์ค์ 
    - ์ธ๋ถ๋ง
        - ์ ์ฒด ๋์ญ : 8080 ํฌํธ ์คํ
        - ๊ด๋ฆฌ๋ง : 22๋ฒ ํฌํธ ์คํ
    - ๋ด๋ถ๋ง
        - ์ธ๋ถ๋ง : 3306 ํฌํธ ์คํ
        - ๊ด๋ฆฌ๋ง : 22๋ฒ ํฌํธ ์คํ
    - ๊ด๋ฆฌ๋ง
        - ์์ ์ ๊ณต์ธ IP : 22๋ฒ ํฌํธ ์คํ
- ์๋ฒ ์์ฑ
    - ์ธ๋ถ๋ง์ ์น ์๋น์ค์ฉ๋์ EC2 ์์ฑ
    - ๋ด๋ถ๋ง์ ๋ฐ์ดํฐ๋ฒ ์ด์ค์ฉ๋์ EC2 ์์ฑ
    - ๊ด๋ฆฌ๋ง์ ๋ฒ ์ค์ณ ์๋ฒ์ฉ๋์ EC2 ์์ฑ
    - ๋ฒ ์ค์ณ ์๋ฒ์ Session Timeout 600s ์ค์ 
    - ๋ฒ ์ค์ณ ์๋ฒ์ Command ๊ฐ์ฌ๋ก๊ทธ ์ค์ 

๐ ์น ์ ํ๋ฆฌ์ผ์ด์ ๋ฐฐํฌ
- ์ธ๋ถ๋ง์ [์น ์ ํ๋ฆฌ์ผ์ด์](https://github.com/next-step/infra-subway-deploy) ์ ๋ฐฐํฌ
- DNS ์ค์ 

[๋ต๋ณ](network.md)

---

### 2. ์๋น์ค ๋ฐฐํฌํ๊ธฐ

โ๏ธ ์๊ตฌ์ฌํญ
- ์ด์ ํ๊ฒฝ ๊ตฌ์ฑํ๊ธฐ
- ๊ฐ๋ฐ ํ๊ฒฝ ๊ตฌ์ฑํ๊ธฐ

์๊ตฌ์ฌํญ ์ค๋ช

๐ ์ด์ ํ๊ฒฝ ๊ตฌ์ฑํ๊ธฐ
- ์น ์ ํ๋ฆฌ์ผ์ด์ ์์ Reverse Proxy ๊ตฌ์ฑํ๊ธฐ
    - ์ธ๋ถ๋ง์ Nginx๋ก Reverse Proxy๋ฅผ ๊ตฌ์ฑ
    - Reverse Proxy์ TLS ์ค์ 
- ์ด์ ๋ฐ์ดํฐ๋ฒ ์ด์ค ๊ตฌ์ฑํ๊ธฐ

๐ ๊ฐ๋ฐ ํ๊ฒฝ ๊ตฌ์ฑํ๊ธฐ
- ์ค์  ํ์ผ ๋๋๊ธฐ
    - JUnit : h2, Local : docker(mysql), Prod : ์ด์ DB๋ฅผ ์ฌ์ฉํ๋๋ก ์ค์ 

[๋ต๋ณ](nginx.conf)

---

### 3. ๋ฐฐํฌ ์คํฌ๋ฆฝํธ ์์ฑํ๊ธฐ

โ๏ธ ์๊ตฌ์ฌํญ
- ๋ฐฐํฌ ์คํฌ๋ฆฝํธ ์์ฑํ๊ธฐ
    - ๋ฐ๋ณต์ ์ผ๋ก ์คํํ๋๋ผ๋ ์ ์์ ์ผ๋ก ๋ฐฐํฌํ๋ ์คํฌ๋ฆฝํธ๋ฅผ ์์ฑํด๋ด๋๋ค.
    - ๊ธฐ๋ฅ ๋จ์๋ก ํจ์๋ก ๋ง๋ค์ด๋ด๋๋ค.
    - ์คํฌ๋ฆฝํธ ์คํ์ ํ๋ผ๋ฏธํฐ๋ฅผ ์ ๋ฌํด๋ด๋๋ค.

[๋ต๋ณ](deploy.sh)
