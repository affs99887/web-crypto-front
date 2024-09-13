# Vue RSA-AES 加密客户端

这是一个使用 Vue 3 的前端应用，演示如何使用浏览器的 Web Cryptography API 进行 RSA 加密、AES-GCM 加密以及与服务器进行交互，最终实现对加密数据的解密操作。

对应的后端：https://github.com/affs99887/web-crypto-backend 或者 https://gitee.com/scape0goat/web-crypto-back

原理文档：https://ucn1x4mi5suo.feishu.cn/wiki/V52pwIxMGiYaPTk73Y4cp8ULn4g?from=from_copylink

## 功能

- 从服务器获取 RSA 公钥并导入到客户端。
- 使用 RSA 公钥加密生成的 AES 密钥。
- 使用 AES-GCM 加密用户输入的数据。
- 将加密后的 AES 密钥、初始化向量（IV）和加密数据发送到服务器。
- 从服务器接收解密后的数据并展示。

## 项目结构

```bash
.
├── public
│   └── index.html
├── src
│   ├── components
│   └── App.vue
├── package.json
└── README.md
```

## 安装

1. 克隆项目到本地：

   ```bash
   git clone <仓库地址>
   cd <项目目录>
   ```

2. 安装依赖：

   ```bash
   npm install
   ```

3. 启动开发服务器：

   ```bash
   npm run dev
   ```

4. 打开浏览器访问 `http://localhost:5173` 查看应用。

## 使用方法

1. 启动前端项目并确保后端服务器（Node.js 加密解密服务）已运行。
2. 在文本框中输入要加密的数据，然后点击“加密并发送”按钮。
3. 前端会生成 AES 密钥，并使用从服务器获取的 RSA 公钥对 AES 密钥进行加密。
4. 使用 AES-GCM 对输入的数据加密后，前端会将加密的 AES 密钥、初始化向量和加密数据发送到服务器。
5. 服务器接收加密的数据，解密后返回明文。
6. 前端展示服务器返回的解密数据。

### 主要函数

- `fetchServerPublicKey`: 获取服务器的 RSA 公钥，并在客户端中导入该公钥。
- `encryptAndSend`: 生成 AES 密钥并对用户输入的数据加密，随后将加密数据发送到服务器。

### 数据加密与发送流程

- 使用 RSA-OAEP 加密 AES 密钥。
- 使用 AES-GCM 加密用户输入的数据。
- 生成并使用一个随机的 12 字节 IV。
- 通过 `fetch` API 将加密的 AES 密钥、IV 和加密数据发送到服务器进行解密。

## 注意事项

- 服务器需要运行在 `http://localhost:3000`，并提供 `/public-key` 和 `/decrypt` 端点。
- 确保后端服务器的 RSA 公钥生成和解密功能正常工作。

## 许可

此项目遵循 MIT 许可协议。详见 [LICENSE](LICENSE) 文件。