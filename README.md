# Linux 系统连接同济大学 VPN 解决方案

同济大学使用 Pulse Secure (Juniper Pulse) 的 VPN 认证服务，本应在 Linux 下支持 [openconnect](https://github.com/openconnect/openconnect) 的连接，但是用`openconnect --juniper vpn.tongji.cn`连接时，在选择 Realm 时会出现这个：

```
GET https://vpn.tongji.cn/dana-na/auth/url_default/welcome.cgi
SSL negotiation with vpn.tongji.cn
Connected to HTTPS on vpn.tongji.cn
frmLogin
校外用户
统一身份认证用户|
                ]:
```

将其导出到文本，是这样的：

```
WARNING: Juniper Network Connect support is experimental.
It will probably be superseded by Junos Pulse support.
frmLogin
realm [
校外用户
                |
统一身份认证用户
                ]:
```

实际上，这一选择提示本应是这样的：

```
realm [校外用户|统一身份认证用户]: # 可在此输入“校外用户”和“统一身份认证用户”
```

但是，由于 [](https://vpn.tongji.cn/dana-na/auth/url_default/welcome.cgi) 源代码中的

```
<select size="1" name="realm">

              <option value="校外用户">
校外用户
                </option>

              <option value="统一身份认证用户">
统一身份认证用户
                </option>

            </select>
```

两个`<option>`中的innerText多了很多空格及换行符，故应该输入的字符串也应完整包含这些空格和换行符，而这是不可能达到的。

故必须绕过这一输入。juniper-vpn-py 用获取网页填写表单获取 DSID token 再传给 openconnect 的方式达到这一点。

这个 Forked Repository 对于 juniper-vpn-py 做了一些改动，使其支持选择带中文 Unicode 字符的 Realm。

## 操作方法

- 满足 `requirements.txt` 中的 pip 包依赖
- 安装 [ocproxy](https://github.com/cernekee/ocproxy) 以将 openconnect 建立的 vpn 转换为 socks5 代理。如果你不需要，请在 `tongji.cfg` 中作修改
- 运行 `python(2) juniper-vpn.py -c tongji.cfg`

# Old Readme

Connecting to a Juniper VPN requires the generation of a DSID token.
Generating this token involves authentication and host checking. The
juniper-vpn.py script performs authentication, and the tncc.py script
performs host checking. The DSID token can then be passed to openconnect.

Alternatively, openconnect can perform the authentication steps on it's own
and utilize the tncc.py script for host checker functionality. This is
the recommended mode of operation. Instructions for using openconnect to
perform authentication and tncc.py to perform host checking are contained in:

README.host_checker

Alternate instructions to allow juniper-vpn.py to perform authentication and
tncc.py to perform host checking are contained in:

README.authenticator

Python dependencies can be found at `requirements.txt` file.
