<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>스마트 단어장</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 400px;
            text-align: center;
            display: none;
            max-height: 90vh;
            overflow-y: auto;
        }
        .active { display: block; }
        h1, h2 { color: #2c3e50; font-size: 1.5em; margin-bottom: 5px;}
        select, input, button {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border-radius: 8px;
            border: 1px solid #ccc;
            font-size: 1em;
            box-sizing: border-box;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }
        button:hover { background-color: #2980b9; }
        .btn-secondary { background-color: #95a5a6; }
        .btn-secondary:hover { background-color: #7f8c8d; }
        .btn-upload { background-color: #e67e22; color: white; display: block; text-align: center; font-weight: bold; }
        .btn-upload:hover { background-color: #d35400; }
        .btn-wrong-manage { background-color: #8e44ad; }
        .btn-wrong-manage:hover { background-color: #732d91; }
        
        #word-display {
            font-size: 2.2em;
            font-weight: bold;
            margin: 20px 0;
            color: #2c3e50;
        }
        .option-btn {
            background-color: #ecf0f1;
            color: #333;
            border: 2px solid #bdc3c7;
            text-align: left;
            padding: 15px;
            line-height: 1.4;
        }
        .option-btn.correct { background-color: #2ecc71; color: white; border-color: #27ae60; }
        .option-btn.wrong { background-color: #e74c3c; color: white; border-color: #c0392b; }
        
        #seed-display-area {
            background: #eee;
            padding: 15px;
            border-radius: 5px;
            word-break: break-all;
            display: none;
            margin-top: 10px;
        }
        #load-status {
            font-size: 0.85em;
            color: #e67e22;
            font-weight: bold;
            margin-bottom: 15px;
        }
        .status-bar {
            display: flex; 
            justify-content: space-between; 
            align-items: center;
            font-size: 0.9em; 
            color: #7f8c8d; 
            border-bottom: 2px solid #eee; 
            padding-bottom: 10px;
        }
        #quiz-progress { font-weight: bold; color: #3498db; font-size: 1.1em; }

        /* 오답 관리 리스트 스타일 */
        #wrong-list-container {
            max-height: 350px; 
            overflow-y: auto; 
            text-align: left; 
            margin-bottom: 20px; 
            border: 1px solid #ddd; 
            border-radius: 8px; 
            padding: 10px; 
            background: #fafafa;
        }
        .wrong-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            border-bottom: 1px solid #eee;
        }
        .wrong-item:last-child { border-bottom: none; }
        .wrong-word-text { font-weight: bold; color: #2c3e50; font-size: 1.1em; }
        .wrong-meaning-text { font-size: 0.9em; color: #7f8c8d; margin-top: 3px; }
        .btn-delete {
            background-color: #ff7675;
            color: white;
            border: none;
            border-radius: 5px;
            width: auto;
            padding: 8px 12px;
            cursor: pointer;
            margin: 0 0 0 10px;
            font-size: 0.9em;
        }
        .btn-delete:hover { background-color: #d63031; }
    </style>
</head>
<body>

    <!-- 메인 화면 -->
    <div id="screen-menu" class="container active">
        <h1>📚 스마트 단어장</h1>
        <div id="load-status">⏳ 드라이브에서 단어장을 불러오는 중...</div>
        
        <select id="day-select">
            <option value="wrong">⚠️ 내 오답 노트 퀴즈 풀기</option>
        </select>
        <button onclick="startQuiz()">학습 시작</button>
        <button class="btn-wrong-manage" onclick="showWrongList()">📝 오답 단어 목록 보기 및 관리</button>

        <hr style="margin: 25px 0; border: 0; border-top: 1px solid #ddd;">

        <h2>📁 외부 TXT 파일 직접 열기</h2>
        <p style="font-size: 0.8em; color: #7f8c8d; margin-top: -10px; margin-bottom: 10px;">(자동 불러오기 실패 시에만 사용하세요)</p>
        <label for="file-input" class="btn-upload" style="padding: 12px; border-radius: 8px; cursor: pointer;">
            수동으로 파일 선택하기
        </label>
        <input type="file" id="file-input" accept=".txt" style="display: none;">

        <hr style="margin: 25px 0; border: 0; border-top: 1px solid #ddd;">
        
        <h2>💾 진행 상황 저장/불러오기</h2>
        <button class="btn-secondary" onclick="generateSeed()">짧은 오답 저장코드(Seed) 발급</button>
        <div id="seed-display-area"></div>

        <input type="text" id="seed-input" placeholder="여기에 시드 코드를 붙여넣으세요">
        <button class="btn-secondary" onclick="loadSeed()">시드 코드로 오답 불러오기</button>
    </div>

    <!-- 퀴즈 화면 -->
    <div id="screen-quiz" class="container">
        <div class="status-bar">
            <span id="quiz-mode-title" style="font-weight: bold; flex: 1; text-align: left;">Day 1</span>
            <span id="quiz-progress" style="flex: 1; text-align: center;">0 / 0</span>
            <span style="flex: 1; text-align: right;">오답 누적: <strong id="wrong-count" style="color:#e74c3c;">0</strong>개</span>
        </div>
        
        <div id="word-display">Loading...</div>
        <div id="options-container"></div>
        
        <button id="next-btn" style="display: none; margin-top: 20px; background-color: #2ecc71;" onclick="nextQuestion()">다음 단어 ➔</button>
        <button class="btn-secondary" style="margin-top: 10px;" onclick="showMenu()">도중에 그만하기</button>
    </div>

    <!-- 오답 관리 화면 -->
    <div id="screen-wrong-list" class="container">
        <h2>📝 내 오답 노트 관리</h2>
        <div style="text-align: right; margin-bottom: 10px;">
            <button onclick="clearAllWrongWords()" style="width: auto; padding: 8px 15px; background-color: #e74c3c; font-size: 0.9em; border-radius: 5px;">전체 일괄 삭제 🗑️</button>
        </div>
        
        <div id="wrong-list-container">
            <!-- 오답 리스트가 자바스크립트에 의해 여기에 들어갑니다 -->
        </div>
        
        <button class="btn-secondary" onclick="showMenu()">메인으로 돌아가기</button>
    </div>

    <script>
        let wordData = [];
        let wrongWords = [];
        let currentQuizList = [];
        let totalWords = 0;
        let currentIndex = 0;
        let currentQuestion;
        let isAnswered = false;

        // 구글 드라이브 텍스트 자동 불러오기
        document.addEventListener('DOMContentLoaded', async () => {
            const statusEl = document.getElementById('load-status');
            try {
                const fileId = '1bfaDWnH1gQSEvfMInAeJxdjdsp00qvfB';
                const driveUrl = `https://drive.google.com/uc?export=download&id=${fileId}`;
                const proxyUrl = `https://api.allorigins.win/raw?url=${encodeURIComponent(driveUrl)}`;
                
                const response = await fetch(proxyUrl);
                if (!response.ok) throw new Error('네트워크 오류');
                const text = await response.text();
                
                parseDataAndInit(text);
                statusEl.textContent = `✅ 단어장 자동 불러오기 완료! (총 ${wordData.length}개)`;
                statusEl.style.color = '#2ecc71';
            } catch (error) {
                console.error(error);
                statusEl.textContent = '❌ 자동 로드 실패. 하단의 파일 선택을 이용해주세요.';
                statusEl.style.color = '#e74c3c';
            }
        });

        // 데이터 파싱 및 셋팅
        function parseDataAndInit(rawDataStr) {
            wordData = rawDataStr.trim().split('\n').map(line => {
                const parts = line.split('|');
                if (parts.length >= 3) {
                    return { day: parseInt(parts[0].trim()), word: parts[1].trim(), meaning: parts[2].trim() };
                }
                return null;
            }).filter(item => item !== null && !isNaN(item.day));

            updateDaySelect();
        }

        function updateDaySelect() {
            const daySelect = document.getElementById('day-select');
            daySelect.innerHTML = '<option value="wrong">⚠️ 내 오답 노트 퀴즈 풀기</option>';
            
            const uniqueDays = [...new Set(wordData.map(item => item.day))].sort((a,b)=>a-b);
            uniqueDays.forEach(day => {
                const option = document.createElement('option');
                option.value = day;
                option.textContent = `Day ${day} 학습하기`;
                daySelect.appendChild(option);
            });
        }

        // TXT 수동 파일 불러오기
        document.getElementById('file-input').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(event) {
                parseDataAndInit(event.target.result);
                if (wordData.length > 0) {
                    alert(`성공적으로 ${wordData.length}개의 단어를 수동으로 불러왔습니다!`);
                    document.getElementById('load-status').textContent = `✅ 수동 파일 적용 완료! (총 ${wordData.length}개)`;
                    document.getElementById('load-status').style.color = '#2ecc71';
                }
                document.getElementById('file-input').value = '';
            };
            reader.readAsText(file);
        });

        // 화면 전환
        function showScreen(screenId) {
            document.querySelectorAll('.container').forEach(el => el.classList.remove('active'));
            document.getElementById(screenId).classList.add('active');
        }
        function showMenu() { showScreen('screen-menu'); updateWrongCount(); }

        // --- 오답 단어장 보기 및 관리 기능 ---
        function showWrongList() {
            showScreen('screen-wrong-list');
            renderWrongList();
        }

        function renderWrongList() {
            const container = document.getElementById('wrong-list-container');
            container.innerHTML = '';

            if (wrongWords.length === 0) {
                container.innerHTML = '<div style="text-align: center; color: #7f8c8d; padding: 30px 10px;">등록된 오답이 없습니다!<br>퀴즈를 풀며 틀린 단어를 모아보세요. 🎉</div>';
                return;
            }

            // 역순으로 렌더링 (최근에 틀린 단어가 위로 오게)
            [...wrongWords].reverse().forEach(wordStr => {
                const wordObj = wordData.find(item => item.word === wordStr);
                const meaning = wordObj ? wordObj.meaning : '뜻을 찾을 수 없음';

                const div = document.createElement('div');
                div.className = 'wrong-item';
                div.innerHTML = `
                    <div style="flex: 1;">
                        <div class="wrong-word-text">${wordStr}</div>
                        <div class="wrong-meaning-text">${meaning}</div>
                    </div>
                    <button class="btn-delete" onclick="removeWrongWord('${wordStr}')">삭제</button>
                `;
                container.appendChild(div);
            });
        }

        function removeWrongWord(wordToRemove) {
            wrongWords = wrongWords.filter(w => w !== wordToRemove);
            updateWrongCount();
            renderWrongList(); // 삭제 후 목록 리스트 새로고침
        }

        function clearAllWrongWords() {
            if (wrongWords.length === 0) return;
            if (confirm("정말 오답 노트를 모두 초기화(전체 삭제) 하시겠습니까?")) {
                wrongWords = [];
                updateWrongCount();
                renderWrongList();
            }
        }
        // ------------------------------------

        // 시드(Seed) 기능
        function generateSeed() {
            if (wrongWords.length === 0) {
                alert("아직 틀린 단어가 없습니다!");
                return;
            }
            const seed = wrongWords.map(word => {
                const index = wordData.findIndex(item => item.word === word);
                return index !== -1 ? index.toString(36) : '';
            }).filter(Boolean).join('.');

            const displayArea = document.getElementById('seed-display-area');
            displayArea.style.display = 'block';
            displayArea.innerHTML = `<strong>아래 코드를 복사해두세요:</strong><br><br>
                <span style="font-size: 1.4em; color: #e74c3c; font-weight: bold; letter-spacing: 2px;">${seed}</span>`;
        }

        function loadSeed() {
            const seedInput = document.getElementById('seed-input').value.trim();
            if (!seedInput) {
                alert("시드 코드를 입력해주세요.");
                return;
            }
            try {
                const indices = seedInput.split('.').map(str => parseInt(str, 36));
                wrongWords = indices.map(index => wordData[index]?.word).filter(Boolean);
                
                if (wrongWords.length > 0) {
                    alert(`${wrongWords.length}개의 오답 기록을 불러왔습니다!`);
                    document.getElementById('seed-input').value = '';
                    updateWrongCount();
                } else {
                    alert("코드에 해당하는 단어를 찾을 수 없습니다.");
                }
            } catch (e) {
                alert("잘못된 시드 코드입니다. 다시 확인해주세요.");
            }
        }

        function updateWrongCount() {
            // 메인 화면과 퀴즈 화면의 카운트를 모두 업데이트
            const countElements = document.querySelectorAll('#wrong-count');
            countElements.forEach(el => el.innerText = wrongWords.length);
        }

        // 퀴즈 시작 및 진행 로직
        function startQuiz() {
            const selectedDay = document.getElementById('day-select').value;
            
            if (wordData.length === 0) {
                alert("아직 단어장이 로드되지 않았습니다. 잠시만 기다려주세요.");
                return;
            }

            if (selectedDay === 'wrong') {
                if (wrongWords.length === 0) {
                    alert("오답 노트가 비어있습니다. 먼저 학습을 진행해주세요!");
                    return;
                }
                currentQuizList = wordData.filter(item => wrongWords.includes(item.word));
                document.getElementById('quiz-mode-title').innerText = "오답 복습 퀴즈";
            } else {
                currentQuizList = wordData.filter(item => item.day == selectedDay);
                document.getElementById('quiz-mode-title').innerText = `Day ${selectedDay}`;
            }

            if (currentQuizList.length === 0) {
                alert("해당 파트에 단어가 없습니다.");
                return;
            }

            currentQuizList = shuffleArray(currentQuizList);
            totalWords = currentQuizList.length;
            currentIndex = 0;

            showScreen('screen-quiz');
            loadQuestion();
        }

        function shuffleArray(array) {
            let arr = [...array];
            for (let i = arr.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
            return arr;
        }

        function loadQuestion() {
            if (currentIndex >= totalWords) {
                alert("🎉 이번 파트의 모든 단어 학습을 완료했습니다!");
                showMenu();
                return;
            }

            isAnswered = false;
            document.getElementById('next-btn').style.display = 'none';
            const optionsContainer = document.getElementById('options-container');
            optionsContainer.innerHTML = '';
            
            document.getElementById('quiz-progress').innerText = `${currentIndex + 1} / ${totalWords}`;
            updateWrongCount();

            currentQuestion = currentQuizList[currentIndex];
            document.getElementById('word-display').textContent = currentQuestion.word;

            let wrongOptions = wordData.filter(item => item.word !== currentQuestion.word);
            wrongOptions = shuffleArray(wrongOptions).slice(0, 3);

            let allOptions = shuffleArray([currentQuestion, ...wrongOptions]);

            allOptions.forEach(option => {
                const button = document.createElement('button');
                button.classList.add('option-btn');
                button.textContent = option.meaning;
                button.onclick = () => checkAnswer(button, option.word === currentQuestion.word);
                optionsContainer.appendChild(button);
            });
        }

        function checkAnswer(selectedBtn, isCorrect) {
            if (isAnswered) return;
            isAnswered = true;

            const allButtons = document.querySelectorAll('.option-btn');
            
            if (isCorrect) {
                selectedBtn.classList.add('correct');
            } else {
                selectedBtn.classList.add('wrong');
                allButtons.forEach(btn => {
                    if (btn.textContent === currentQuestion.meaning) btn.classList.add('correct');
                });
                
                if (!wrongWords.includes(currentQuestion.word)) {
                    wrongWords.push(currentQuestion.word);
                }
            }
            
            updateWrongCount();
            document.getElementById('next-btn').style.display = 'block';
        }

        function nextQuestion() {
            currentIndex++;
            loadQuestion();
        }
    </script>
</body>
</html>
