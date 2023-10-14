# CF_Worker
```
import { Ai } from './vendor/@cloudflare/ai.js';

export default {
    async fetch(request, env) {
        async function get_msg_by_ai(MSG) {
            let result = await ai.run('@cf/meta/llama-2-7b-chat-int8', MSG);
            return { role: 'system', content: result.response }
        }
        // 获取用户对的访问全部的URL
        const url = new URL(request.url);
        let userContent = url.searchParams.get('content');

        const tasks = [];
        const ai = new Ai(env.AI);
        let response;
        let chat = {
            messages: [
                { role: 'system', content: '你是一个有好的助手,且回答全部是中文!' },
                { role: 'user', content: userContent },
            ]
        };

        response = await get_msg_by_ai(chat)
        chat.messages.push(response);
        chat.messages.push({role: "user",content: response.content + "继续回答"});
        response = await get_msg_by_ai(chat)
        chat.messages.push(response);
        // chat.messages.push({role: "user",content: response.content + "继续回答"});
        // response = await get_msg_by_ai(chat)
        // chat.messages.push(response);

        // 新的数组用于存储 "role": "system" 的消息
        let systemMessages = [];
        chat.messages.forEach(function(item) {
            if (item.role === "system") {
                systemMessages.push(item);
            }
        });

        // 去掉第一个数组
        systemMessages.shift();
        tasks.push(systemMessages);


        // 添加请求头以消除同源策略的影响
        const responseHeaders = new Headers();
        responseHeaders.set('Access-Control-Allow-Origin', '*');
        responseHeaders.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS'); 
        responseHeaders.set('Access-Control-Allow-Headers', 'Content-Type');

         // 返回特定的数据
         const responseBody = JSON.stringify(tasks);

         return new Response(responseBody, {
             headers: responseHeaders,
             status: 200, // 设置响应状态码
             statusText: 'OK' // 设置响应状态文本
         });
    }

};

```
