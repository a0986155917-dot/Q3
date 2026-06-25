
# 1. 資料庫模型 (models.py)
介紹：定義圖書館的「借閱紀錄」資料表。包含借閱人、書籍、借出/應還日期，並透過 Python 的 `is_overdue` 計算書籍是否已經過期。
```python
from django.db import models
 from django.contrib.auth.models import User
 from datetime import date

 class BorrowRecord(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="借閱人")
    book = models.ForeignKey('Book', on_delete=models.CASCADE, verbose_name="書籍")
    borrow_date = models.DateField(auto_now_add=True, verbose_name="借出日期")
    due_date = models.DateField(verbose_name="應還日期")
    return_date = models.DateField(null=True, blank=True, verbose_name="實際歸還日期")

    @property
    def is_overdue(self):
        if not self.return_date and date.today() > self.due_date:
            return True
        return False

    def __str__(self):
        return f"{self.user.username} 借了 {self.book.title}"
```
# 2. AI 智慧客服核心 (chatbot.py)
介紹：本專案的 RAG 核心大腦。負責讀取本地的圖書館規章（library_rules.txt），並將規章作為背景知識餵給 Gemini API，實現 24 小時精準自動回覆。
```python
import google.genai as genai

def ask_library_chatbot(user_question):
    # 1. 讀取本地圖書館規範文字檔 (RAG 背景知識)
    try:
        with open('library_rules.txt', 'r', encoding='utf-8') as f:
            rules_context = f.read()
    except FileNotFoundError:
        rules_context = "暫無特定圖書館規範。"

    # 2. 初始化 Gemini 客戶端
    client = genai.Client(api_key="YOUR_GEMINI_API_KEY")
    
    # 3. 組合 Prompt，限制 AI 只能根據規章回答
    prompt = f"""
    你現在是電子圖書館的智能客服機器人。
    請嚴格根據以下提供的【圖書規範背景知識】來回答使用者的問題。
    如果問題在背景知識中找不到答案，請禮貌回答「抱歉，這部分我需要幫您轉接人工客服」。

    【圖書規範背景知識】：
    {rules_context}

    使用者問題：{user_question}
    AI 客服回答：
    """
    
    # 4. 呼叫大語言模型 (使用 2.5-flash)
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=prompt,
    )
    
    return response.text
```
# 3.客服前端網頁 (chatbot.html)
簡短介紹：精美的對話框前端網頁 UI。利用 JavaScript 監聽「發送」按鈕，透過 AJAX (Fetch API) 與 Django 後端進行非同步異步通訊，讓使用者免刷新網頁即可與 AI 機器人即時對話。
```HTML
<!-- 簡化版對話框核心架構 -->
<div class="chat-container">
    <div class="chat-header"> 電子圖書館 - AI 智能客服</div>
    <div id="chat-box" class="chat-box">
        <div class="bot-msg">您好！我是圖書館 AI 助手，請提問任何關於借還書規章的問題。</div>
    </div>
    <div class="input-area">
        <input type="text" id="user-input" placeholder="請輸入您的問題...">
        <button id="send-btn">發送</button>
    </div>
</div>

<script>
document.getElementById('send-btn').addEventListener('click', function() {
    let input = document.getElementById('user-input');
    let query = input.value.trim();
    if(!query) return;

    // 顯示使用者的訊息與「思索中...」
    appendMessage(query, 'user');
    let thinking = appendMessage('思索中...', 'bot');

    // 串接後端 Django 路由傳遞問題
    fetch('/api/chat/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ question: query })
    })
    .then(res => res.json())
    .then(data => {
        thinking.innerText = data.answer; // 將思考換成 AI 的真正回覆
    });
    input.value = '';
});
</script>
```
