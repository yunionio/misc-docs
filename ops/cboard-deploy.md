# 部署 cboard

使用 ansible 部署 cboard，确保环境安装了 ansible，并配置了本地免密码 ssh 登录。

## 配置

```bash
# 创建 cboard 部署目录
mkdir -p cboard-deploy/roles
cd cboard-deploy

# clone ansible-cboard playbook
git clone https://github.com/yunionio/ansible-cboard roles/cboard

# 编写部署配置
vim hosts
[cboard]
localhost ansible_user=root

vim playbook.yml
- name: configure cboard
  hosts: cboard
  roles:
  - cboard
  vars:
    port: 8082

# 执行部署
ansible-playbook -i hosts playbook.yml
```

## 访问

之前配置默认会把 cboard 使用 docker 以 hostNetwork 的方式启动到 8082 端口。

```bash
docker ps | grep cboard
dcfa78ba8754        registry.cn-beijing.aliyuncs.com/yunionio/cboard:0.4.1   "/root/startup.sh"
```

访问地址为 `http://IP:8082/cboard/`
