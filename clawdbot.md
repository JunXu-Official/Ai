# 在 Windows 上安装和配置 ClawdBot(OpenClaw)

## 步骤一：参考[claude.md 安装教程](./claude.md)安装Node.js和Git

## 步骤二：安装ClawdBot(OpenClaw)

1. 管理员打开cmd窗口，运行
    ```bash
    npm install -g clawdbot
    ```

2. 安装完成后，验证：
    ```bash
    clawdbot --version
    ```
    输出版本号表示安装成功。

3. 配置ClawdBot(OpenClaw)
    
    - 启动clawdbot
    ```bash
    clawdbot onboard
    ```
    出现"ClawdBot(OpenClaw) is running"表示启动成功。
    ![1.png](./clawdbot_images/1.png)

    - 一直按需配置即可
    ![2.png](./clawdbot_images/2.png)

    - 出现需要具体set的项，直接选择No或者skip for now!

4. 配置完成后，关闭cmd，重新管理员身份打开，运行
    ```bash
    clawdbot gateway --port 18789
    ```
    启动后，在浏览器中输入[http://127.0.0.1:18789/]

5. 潜在问题
    - 出现disconnected (1008): unauthorized: gateway token mismatch (open a tokenized dashboard URL or paste token in Control UI settings)
    解决方法：
        - 到"C:\Users\Lenovo\.clawdbot"找到clawdbot.json文件中找到18789对应的token将其复制；
        - 在[http://127.0.0.1:18789/]中点击Overview，然后将token粘贴到Gateway Token中,点击connect.
