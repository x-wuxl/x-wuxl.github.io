---
layout: post
title:  "全自动获取免费机场订阅节点"

---

### 前置条件：
- python3.9及以上环境

### 本地:
1. git clone https://github.com/wzdnzd/aggregator.git
2. pip3 install pyYAML tqdm requests
3. cd aggregator
4. python -u subscribe/collect.py -si
5. 运行完后在项目目录下./data/subscribes.txt 就是跑完的所有机场订阅链接

### 服务器:
1. 新建一个 gist 并指定名称和后缀:configsub.yaml   
[Create a new Gist](https://gist.github.com/)
2. 新增一个 github token   [Developer Settings](https://github.com/settings/tokens?type=beta)  
3. git clone https://github.com/wzdnzd/aggregator.git
4. pip3 install pyYAML tqdm requests
5. cd aggregator
6. python -u subscribe/collect.py -si
7. 在项目目录aggregator/subscribe下添加脚本merged2upload.py，
8. python subscribe/merged2upload.py
9. 执行完成后，在 gist 中点击 Raw，该链接就是你的订阅连接了

merged2upload.py 是将第6步跑出的所有机场订阅链接解析并合并上传到 gist 中

merged2upload.py 代码，需要将代码中的 gist_id、github_token 替换成你的
```
import requests
import base64
import os

def fetch_and_decode_base64(url):
    print(url)
    try:
        response = requests.get(url)
        response.raise_for_status()
        decoded_content = base64.b64decode(response.text)
        return decoded_content.decode('utf-8')
    except requests.RequestException as e:
        print(f"Error fetching {url}: {e}")
        return None

def upload_to_gist(content, gist_id, github_token):
    url = f"https://api.github.com/gists/{gist_id}"
    headers = {
        'Authorization': f'token {github_token}',
        'Accept': 'application/vnd.github.v3+json'
    }
    data = {
        "files": {
            "configsub.yaml": {
                "content": content
            }
        }
    }
    try:
        response = requests.patch(url, headers=headers, json=data)
        response.raise_for_status()
        print(f"Successfully updated Gist: {gist_id}")
    except requests.RequestException as e:
        print(f"Error updating Gist: {e}")

def main():
    current_dir = os.path.dirname(__file__)
    parent_dir = os.path.dirname(current_dir)
    file_path = os.path.join(parent_dir, 'data', 'subscribes.txt')
    
    with open(file_path, 'r') as file:
        urls = file.read().strip().split('\n')

    all_decoded_texts = []

    for url in urls:
        decoded_content = fetch_and_decode_base64(url)
        if decoded_content:
            all_decoded_texts.append(decoded_content)

    merged_content = "\n".join(all_decoded_texts)
    encoded_merged_content = base64.b64encode(merged_content.encode('utf-8')).decode('utf-8')

    merged_file_path = os.path.join(parent_dir, 'data', 'merged.txt')
    with open(merged_file_path, 'w') as file:
        file.write(encoded_merged_content)
        print(f"Encoded merged content written to {merged_file_path}")

    # Upload the merged content to the Gist 
    github_token = 'your github_token'
    gist_id = 'your gist_id'
    upload_to_gist(encoded_merged_content, gist_id, github_token)

if __name__ == "__main__":
    main()
```

