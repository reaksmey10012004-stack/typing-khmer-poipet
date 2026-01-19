<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Khmer/English Typing Test Pro</title>
    <style>
        :root {
            --primary-color: #4a90e2;
            --success-color: #2ecc71;
            --error-color: #e74c3c;
            --bg-color: #f0f2f5;
        }

        body {
            font-family: 'Kantumruy Pro', 'Segoe UI', Tahoma, sans-serif;
            background-color: var(--bg-color);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        .typing-container {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            width: 90%;
            max-width: 700px;
            text-align: center;
        }

        .stats-bar {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
            font-size: 1.2rem;
            font-weight: bold;
            color: #444;
        }

        .stat-box {
            background: #f8f9fa;
            padding: 10px 20px;
            border-radius: 10px;
            border: 1px solid #ddd;
        }

        #quote-display {
            font-size: 1.4rem;
            line-height: 1.6;
            margin-bottom: 20px;
            text-align: left;
            background: #fff;
            padding: 20px;
            border-radius: 10px;
            border-left: 5px solid var(--primary-color);
            min-height: 80px;
            word-wrap: break-word;
        }

        .char-correct { color: var(--success-color); }
        .char-incorrect { 
            color: var(--error-color); 
            background: #ffdada;
            border-radius: 2px;
        }
        .char-current {
            border-bottom: 2px solid var(--primary-color);
        }

        #input-field {
            width: 100%;
            height: 120px;
            padding: 15px;
            font-size: 1.1rem;
            border: 2px solid #ddd;
            border-radius: 10px;
            outline: none;
            transition: 0.3s;
            resize: none;
        }

        #input-field:focus {
            border-color: var(--primary-color);
            box-shadow: 0 0 8px rgba(74, 144, 226, 0.3);
        }

        .btn-reset {
            margin-top: 20px;
            padding: 12px 30px;
            font-size: 1rem;
            background: var(--primary-color);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: 0.3s;
        }

        .btn-reset:hover { background: #357abd; }
        
        /* បិទមិនឱ្យ Copy អត្ថបទ */
        #quote-display { user-select: none; }
    </style>
</head>
<body>

<div class="typing-container">
    <h2>Typing Test Pro</h2>
    
    <div class="stats-bar">
        <div class="stat-box">ពេល: <span id="timer">60</span>s</div>
        <div class="stat-box">WPM: <span id="wpm">0</span></div>
        <div class="stat-box">កំហុស: <span id="errors">0</span></div>
    </div>

    <div id="quote-display">កំពុងទាញយកអត្ថបទ...</div>
    
    <textarea id="input-field" placeholder="ចាប់ផ្តើមវាយនៅទីនេះ ដើម្បីចាប់ផ្តើមនាឡិកា..." autofocus></textarea>
    
    <br>
    <button class="btn-reset" onclick="resetGame()">ចាប់ផ្តើមឡើងវិញ</button>
</div>

<script>
    const quoteDisplay = document.getElementById('quote-display');
    const inputField = document.getElementById('input-field');
    const timerElement = document.getElementById('timer');
    const wpmElement = document.getElementById('wpm');
    const errorElement = document.getElementById('errors');

    let timeLeft = 60;
    let timerInterval = null;
    let isStarted = false;
    let totalWordsTyped = 0;

    const quotes = [
        "tang pi kmeang mor nh jol jit leang ey muy derl mak pa ham rhot knom min derl sdab pamak rbos knom te knom teang tea due dak kert pel thom dg kdey terb dg tha kert brab jong oy kon laor ter kon nis vea babsa tang pi doch dol thom nh min derl jes klach ah na jivit bang thom poipet pit chea kmeang ban ka puk nh kmeang dai sor puk ah na jg klang jol mor bang oy tang yes ser you nob ah na vea tha klang anh jmer ah na hean lok yes  ser you nob ah anh kmeang poipet ah jg lg jol mor anh mean ter pi nak puk mak anh te ah sak zin to bc zin neak anh mean ter pi neak ber yg sl mnus ng hx yg trov plok vea sit ban dg ber min chgay jam bos jol  .",
        "chea kmeang ban ka puk nh kmeang dai sor puk ah na jg klang jol mor bang oy tang yes ser you nob ah na.",
        "poipet ah jg lg jol mor anh mean ter pi nak puk mak anh te ah sak zin to bc zin neak anh mean ter pi neak ber yg sl mnus ng hx yg trov plok vea sit ban dg ber min chgay jam bos.",
        "ah jm lana hg work doch ach yg has muy web leler jeang ke poipet ah jg lg jol mor anh mean ter pi nak puk mak anh te ah sak zin to bc zin neak anh mean ter pi neak ber yg sl mnus ng hx yg trov plok vea sit ban dg ber min chgay jam bos.",
        "anh hot ng hg na ah lana ah sles ah jm pi neak ber yg sl mnus ng hx yg trov plok vea sit ban dg ber min chgay jam bos jol.",
        "mean ter bang kev chanreaksmey te smos jeang ke muy poipet ."
    ];

    function loadNewQuote() {
        const randomIndex = Math.floor(Math.random() * quotes.length);
        const quote = quotes[randomIndex];
        quoteDisplay.innerHTML = '';
        quote.split('').forEach(char => {
            const span = document.createElement('span');
            span.innerText = char;
            quoteDisplay.appendChild(span);
        });
        inputField.value = '';
    }

    inputField.addEventListener('input', () => {
        if (!isStarted) {
            startTimer();
            isStarted = true;
        }

        const arrayQuote = quoteDisplay.querySelectorAll('span');
        const arrayValue = inputField.value.split('');
        
        let errors = 0;
        let isAllCorrect = true;

        arrayQuote.forEach((charSpan, index) => {
            const charTyped = arrayValue[index];

            if (charTyped == null) {
                charSpan.classList.remove('char-correct', 'char-incorrect', 'char-current');
                if (index === arrayValue.length) charSpan.classList.add('char-current');
                isAllCorrect = false;
            } else if (charTyped === charSpan.innerText) {
                charSpan.classList.add('char-correct');
                charSpan.classList.remove('char-incorrect', 'char-current');
            } else {
                charSpan.classList.add('char-incorrect');
                charSpan.classList.remove('char-correct', 'char-current');
                errors++;
                isAllCorrect = false;
            }
        });

        errorElement.innerText = errors;

        // បើវាយចប់ឃ្លា និងត្រឹមត្រូវទាំងអស់ ឱ្យលោតទៅឃ្លាថ្មីភ្លាម
        if (isAllCorrect && arrayValue.length === arrayQuote.length) {
            totalWordsTyped += arrayQuote.length / 5; // ប្រហែលជា ៥ តួអក្សរក្នុង ១ ពាក្យ
            loadNewQuote();
            updateWPM();
        }
    });

    function startTimer() {
        timerInterval = setInterval(() => {
            if (timeLeft > 0) {
                timeLeft--;
                timerElement.innerText = timeLeft;
                updateWPM();
            } else {
                gameOver();
            }
        }, 1000);
    }

    function updateWPM() {
        const timePassed = (60 - timeLeft) / 60;
        if (timePassed > 0) {
            const currentWPM = Math.round(totalWordsTyped / timePassed);
            wpmElement.innerText = currentWPM;
        }
    }

    function gameOver() {
        clearInterval(timerInterval);
        inputField.disabled = true;
        alert(`ចប់ម៉ោង! ល្បឿនរបស់អ្នកគឺ: ${wpmElement.innerText} WPM`);
    }

    function resetGame() {
        clearInterval(timerInterval);
        timeLeft = 60;
        isStarted = false;
        totalWordsTyped = 0;
        timerElement.innerText = timeLeft;
        wpmElement.innerText = 0;
        errorElement.innerText = 0;
        inputField.disabled = false;
        inputField.value = '';
        loadNewQuote();
        inputField.focus();
    }

    // ចាប់ផ្តើមហ្គេមដំបូង
    loadNewQuote();
</script>

</body>
</html>
