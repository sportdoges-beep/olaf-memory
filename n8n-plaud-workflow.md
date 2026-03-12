# n8n Workflow: Plaud → Telegram + GitHub

## Триггер

**Webhook (Zapier)**
- URL: `https://your-n8n-instance.com/webhook/plaud`
- Method: POST

## Nodes

### 1. Webhook
```
Input: Plaud → Zapier (text/plain)
```

### 2. Code (Очистка текста)
```javascript
// Удаляем шумные транскрипты
let text = $input.first().json.text;
text = text.replace(/\[музыка\]/gi, '');
text = text.replace(/\[шум\]/gi, '');
text = text.replace(/\[неразборчиво\]/gi, '');
return { text: text.trim() };
```

### 3. Date Format
```javascript
const now = new Date();
return {
  date: now.toISOString().split('T')[0],
  filename: `plaud/${now.toISOString().split('T')[0]}/${now.getTime()}.md`
};
```

### 4. GitHub (Create File)
- Repository: `sportdoges-beep/olaf-memory`
- Branch: `main`
- Path: `{{ $json.filename }}`
- Content: `{{ $json.content }}`
- Commit Message: `Add Plaud note {{ $json.date }}`

### 5. Telegram (Send Message)
- Chat ID: your_chat_id
- Text: `📝 Новая заметка:

{{ $json.text }}`

## Альтернатива: GitHub API напрямую

Если без n8n — можно использовать GitHub API:

```bash
curl -X POST https://api.github.com/repos/sportdoges-beep/olaf-memory/contents/plaud/${date}.md \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"message": "Add Plaud note", "content": "BASE64_CONTENT"}'
```

## Настройка Zapier → n8n

1. Создай webhook в n8n
2. В Zapier: Plaud → Webhook (POST)
3. Отправь тестовую заметку
4. Проверь GitHub
