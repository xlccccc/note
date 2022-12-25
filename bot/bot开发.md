```json
私聊格式：
{'post_type': 'message', 'message_type': 'private', 'time': 1660807830, 'self_id': 663468796, 'sub_type': 'friend', 'user_id': 2595251998, 'target_id': 663468796, 'message': '你好', 'raw_message': '你好', 'font': 0, 'sender': {'age': 0, 'nickname': '☁', 'sex': 'unknown', 'user_id': 2595251998}, 'message_id': -284131328}
群聊格式：
{'post_type': 'message', 'message_type': 'group', 'time': 1660807752, 'self_id': 663468796, 'sub_type': 'normal', 'sender': {'age': 0, 'area': '', 'card': '', 'level': '', 'nickname': '☁', 'role': 'owner', 'sex': 'unknown', 'title': '', 'user_id': 2595251998}, 'user_id': 2595251998, 'group_id': 531222907, 'font': 0, 'message': '你好', 'message_seq': 12258, 'raw_message': '你好', 'message_id': 2075067185, 'anonymous': None}

{'send_msg': {'message_type': 'private', 'user_id': '2595251998', 'message': '你78谁啊'}}

{'post_type': 'message', 'message_type': 'group', 'time': 1660830200, 'self_id': 663468796, 'sub_type': 'normal', 'anonymous': None, 'group_id': 531222907, 'raw_message': '[CQ:at,qq=663468796] !enable', 'sender': {'age': 0, 'area': '', 'card': '', 'level': '', 'nickname': '☁', 'role': 'owner', 'sex': 'unknown', 'title': '', 'user_id': 2595251998}, 'message_id': 2053990662, 'font': 0, 'message': '[CQ:at,qq=663468796] !enable', 'message_seq': 12358, 'user_id': 2595251998}
```

```py
import requests
import json
msg = ['[CQ:image,file=http://1.15.107.150:7778/159.jpg,type=show,id=40004]', '123']
group_msg = []
for item in msg:
    each_msg = {
        "type": "none",
        "data": {
            "name": '123',
            "uin": '2595251998',
            "content": item
        }
    }
    group_msg.append(each_msg)
data = {
    'group_id':531222907, # '消息发送的QQ群号'
    'messages':group_msg
}
cq_url = "http://127.0.0.1:5700/send_group_forward_msg"
rev3 = requests.post(cq_url,json=data)
```

