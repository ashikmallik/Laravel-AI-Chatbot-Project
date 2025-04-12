# Laravel AI Chatbot Project (Fontend Part)

এই প্রজেক্টটি **Laravel** ব্যবহার করে একটি AI-চালিত **Chatbot** তৈরি করার জন্য, যা **OpenAI API** ব্যবহার করে ইউজারের প্রশ্নের উত্তর দেয়।

## 🛠️ ব্যবহৃত টেকনোলজি:
- **Laravel** (PHP Framework)
- **Livewire** (Real-time Interactivity)
- **OpenAI API** (AI-powered responses)

## ⚙️ ডিপ্লয়মেন্ট ইনস্ট্রাকশন:

### ১. **Laravel Install করা:**
প্রথমে প্রজেক্ট ডাউনলোড করে নিচের কমান্ডটি চালিয়ে **composer** দিয়ে প্যাকেজ গুলো ইন্সটল করতে হবে।

```
composer install
```

### ২. .env ফাইলে API Key সেট করা:
এবার OpenAI API Key তোমার .env ফাইলে সেট করতে হবে। তুমি তোমার API Key OpenAI থেকে পেয়ে .env ফাইলে এইভাবে সেট করা ।
```
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

```
## ৩. Livewire ইন্সটল করা:
Livewire ইন্সটল করতে নিচের কমান্ড ব্যবহার ।
```
composer require livewire/livewire
```
👉 তারপর blade layout ফাইলে <head>-এ:
```
@livewireStyles
...
</head>
<body>
    ...
    @livewireScripts
</body>
```
# 4. Livewire Component তৈরি করো
```
php artisan make:livewire AiChat
```
# 5. app/Http/Livewire/AiChat.php ফাইল আপডেট :
```
namespace App\Http\Livewire;

use Livewire\Component;
use App\Services\AIHelper;

class AiChat extends Component
{
    public $message = '';
    public $chatHistory = [];

    public function sendMessage()
    {
        if (trim($this->message) === '') return;

        $userMessage = $this->message;
        $this->chatHistory[] = ['user' => $userMessage];

        $ai = new AIHelper();
        $reply = $ai->chat($userMessage);

        $this->chatHistory[] = ['ai' => $reply];
        $this->message = '';
    }

    public function render()
    {
        return view('livewire.ai-chat');
    }
}
```

# 6. resources/views/livewire/ai-chat.blade.php ফাইল :
```
<div class="max-w-xl mx-auto p-4 bg-white shadow-xl rounded-xl">
    <h2 class="text-xl font-bold mb-4">💬 AI Chatbot</h2>

    <div class="h-64 overflow-y-auto mb-4 space-y-2">
        @foreach ($chatHistory as $chat)
            @if (isset($chat['user']))
                <div class="text-right">
                    <span class="inline-block bg-blue-100 text-blue-800 px-4 py-2 rounded-lg">
                        {{ $chat['user'] }}
                    </span>
                </div>
            @elseif (isset($chat['ai']))
                <div class="text-left">
                    <span class="inline-block bg-gray-100 text-gray-800 px-4 py-2 rounded-lg">
                        🤖 {{ $chat['ai'] }}
                    </span>
                </div>
            @endif
        @endforeach
    </div>

    <form wire:submit.prevent="sendMessage" class="flex space-x-2">
        <input type="text" wire:model="message" class="flex-1 border rounded px-3 py-2" placeholder="Type your message...">
        <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded">Send</button>
    </form>
</div>
```
যেকোন Blade ফাইলে এটা যুক্ত করা
```
@livewire('ai-chat')
```
# 🧪 রেজাল্ট:
ইউজার প্রশ্ন করবে

AI (OpenAI) সেই প্রশ্নের উত্তর দিবে real-time

Chat history দেখাবে উপরে সুন্দরভাবে ✅


# Laravel AI Chatbot Project (Backend Part)
app/Services/AIHelper.php ফাইল তৈরি করতে হবে ।
```
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;

class AIHelper
{
    protected $apiKey;
    protected $baseUrl;

    public function __construct()
    {
        $this->apiKey = env('OPENAI_API_KEY');
        $this->baseUrl = 'https://api.openai.com/v1';
    }

    // 🧠 General Chat (Chatbot এর জন্য)
    public function chat(string $message)
    {
        $response = Http::withToken($this->apiKey)->post("{$this->baseUrl}/chat/completions", [
            'model' => 'gpt-3.5-turbo',
            'messages' => [
                ['role' => 'user', 'content' => $message],
            ],
            'temperature' => 0.7,
        ]);

        return $response['choices'][0]['message']['content'] ?? 'Sorry, কিছু ভুল হয়েছে।';
    }

    // 🔍 Search Input Embedding (Smart search এর জন্য)
    public function getEmbedding(string $text)
    {
        $response = Http::withToken($this->apiKey)->post("{$this->baseUrl}/embeddings", [
            'model' => 'text-embedding-ada-002',
            'input' => $text,
        ]);

        return $response['data'][0]['embedding'] ?? null;
    }

    // ✅ Review Sentiment Analysis
    public function analyzeSentiment(string $text)
    {
        $prompt = "এই রিভিউটা positive না negative?: \"$text\"";

        $response = Http::withToken($this->apiKey)->post("{$this->baseUrl}/completions", [
            'model' => 'text-davinci-003',
            'prompt' => $prompt,
            'max_tokens' => 10,
        ]);

        return strtolower(trim($response['choices'][0]['text']));
    }
}
```
# Controller
```
php artisan make:controller AIController
```
# AIController.php
```
use App\Services\AIHelper;

public function chat(Request $request, AIHelper $ai)
{
    $reply = $ai->chat($request->input('message'));
    return response()->json(['reply' => $reply]);
}

public function reviewSentiment(Request $request, AIHelper $ai)
{
    $sentiment = $ai->analyzeSentiment($request->input('review'));
    return response()->json(['sentiment' => $sentiment]);
}

```
# Routes যুক্ত করা (routes/web.php বা api.php)
```
Route::post('/ai/chat', [AIController::class, 'chat']);
Route::post('/ai/review-sentiment', [AIController::class, 'reviewSentiment']);

