<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <title>제주코딩베이스캠프 게스트하우스 RAG 챗봇</title>
    <style>
        body {
            background-color: #1A1B1E;
            color: #ECECF1;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
        }
        .header {
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0;
            margin-bottom: 2rem;
            background-color:rgba(250, 150, 57, 0.8);
            position: relative;
            height: 200px;
            overflow: hidden;
        }
        .logo {
            display: none;
        }
        .main-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            max-width: 900px;
            margin: 0 auto;
            padding: 0 1rem;
            width: 100%;
        }
        .chat-container {
            flex: 1;
            margin-bottom: 100px;
        }
        .message-group {
            margin-bottom: 1.5rem;
        }
        .user-message, .assistant-message {
            max-width: 100%;
            margin: 0 auto;
            padding: 1rem;
            display: flex;
            gap: 1rem;
            line-height: 1.6;
            border-radius: 8px;
            position: relative;
        }
        .user-message {
            background-color: #2D2E35;
            margin-left: 2rem;
            margin-right: 1rem;
        }
        .assistant-message {
            background-color: #1E1F23;
            margin-right: 2rem;
            margin-left: 1rem;
        }
        .message-content {
            flex: 1;
            overflow-wrap: break-word;
            font-size: 0.95rem;
            color: #E4E4E7;
        }
        .message-avatar {
            width: 28px;
            height: 28px;
            border-radius: 4px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 14px;
            font-weight: 600;
            flex-shrink: 0;
        }
        .user-avatar {
            background-color: #5436DA;
        }
        .assistant-avatar {
            background-color: #19C37D;
        }
        .input-container {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            padding: 2rem;
            background: linear-gradient(to bottom, transparent, #1A1B1E 20%);
        }
        .input-group {
            max-width: 768px;
            margin: 0 auto;
            background-color: #27282C;
            border-radius: 12px;
            padding: 0.75rem;
            display: flex;
            align-items: center;
            border: 1px solid #383838;
        }
        .form-control {
            background-color: transparent;
            border: none;
            color: #ECECF1;
            padding: 0.5rem 1rem;
            font-size: 0.95rem;
            line-height: 1.5;
            flex: 1;
        }
        .form-control:focus {
            outline: none;
            box-shadow: none;
            background-color: transparent;
            color: #ECECF1;
        }
        .form-control::placeholder {
            color: #9CA3AF;
        }
        .btn-group {
            display: flex;
            gap: 0.5rem;
            padding: 0 0.5rem;
        }
        .btn-icon {
            background-color: transparent;
            border: none;
            color: #9CA3AF;
            padding: 0.5rem;
            cursor: pointer;
            transition: all 0.2s;
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .btn-icon:hover {
            background-color: #383838;
            color: #ECECF1;
        }
        .btn-icon svg {
            width: 20px;
            height: 20px;
        }
        @media (max-width: 768px) {
            .input-container {
                padding: 1rem;
            }
            .main-container {
                padding: 0 0.5rem;
            }
        }
        
        .loading-message {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        
        .loading-dots {
            display: flex;
            gap: 0.3rem;
        }
        
        .loading-dot {
            width: 8px;
            height: 8px;
            background-color: #E4E4E7;
            border-radius: 50%;
            animation: loading 1.4s infinite;
        }
        
        .loading-dot:nth-child(2) {
            animation-delay: 0.2s;
        }
        
        .loading-dot:nth-child(3) {
            animation-delay: 0.4s;
        }
        
        @keyframes loading {
            0%, 100% {
                opacity: 0.4;
                transform: scale(0.8);
            }
            50% {
                opacity: 1;
                transform: scale(1);
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <div class="header-overlay"></div>
        <div class="header-content">
            <h1 style="color: white; font-size: 2.5rem; margin-bottom: 1rem;">제주도 맛집 추천</h1>
            <p style="color: white; font-size: 1.2rem; opacity: 0.9;">현지인이 추천하는 진짜 맛집을 찾아보세요</p>
        </div>
    </div>
    <div class="main-container">
        <div class="chat-container" id="chat-box"></div>
        <div class="input-container">
            <div class="input-group">
                <input type="text" id="user-input" class="form-control" placeholder="무엇이든 물어보세요">
                <div class="btn-group">
                    <button class="btn-icon">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M12 18V6"></path>
                            <path d="M6 12H18"></path>
                        </svg>
                    </button>
                    <button id="send-button" class="btn-icon">
                        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M22 2L11 13"></path>
                            <path d="M22 2L15 22L11 13L2 9L22 2Z"></path>
                        </svg>
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        const documents = [
            { id: 1, text: "제주시 연동에 위치한 '제주 흑돼지 연동'은 30년 전통의 흑돼지 전문점입니다. 특히 흑돼지 오겹살과 목살이 유명하며, 제주 현지인들도 자주 찾는 맛집입니다. 가격은 흑돼지 오겹살 200g 기준 3만원대입니다. 특제 된장소스와 함께 구워 먹는 흑돼지가 일품이며, 제주 향토 음식인 몸국도 함께 주문할 수 있습니다. 영업시간은 11:30~22:00이며, 연중무휴로 운영됩니다. 주차장이 완비되어 있고 단체석도 있어 가족 모임이나 회식에도 좋습니다." },
            
            { id: 2, text: "서귀포시 중문관광단지 근처의 '중문 갈치조림'은 제주 갈치요리 전문점입니다. 갈치조림과 갈치구이가 대표메뉴이며, 신선한 제주 갈치만 사용합니다. 갈치조림 대자 기준 4만원대입니다. 갈치조림은 제주산 은갈치를 사용하며, 특제 양념과 함께 푸짐한 야채를 곁들여 조립니다. 식사 메뉴로 갈치구이정식(3만원)도 인기가 많습니다. 영업시간은 10:00~21:00이며, 매주 화요일은 휴무입니다. 중문관광단지에서 도보 5분 거리에 위치해 있어 접근성이 좋습니다." },
            
            { id: 3, text: "제주시 구좌읍에 있는 '성산 해녀식당'은 해녀들이 직접 잡은 해산물로 요리를 제공합니다. 전복죽, 소라무침, 해삼물회 등이 유명하며, 전복죽 한 그릇이 1만5천원입니다. 해녀들이 직접 잡은 신선한 해산물을 사용하여 싱싱함이 특징이며, 성산일출봉이 보이는 오션뷰를 자랑합니다. 계절별로 다양한 해산물 메뉴가 준비되어 있으며, 특히 보말칼국수(1.2만원)와 해녀물회(2만원)가 여름철 인기메뉴입니다. 영업시간은 8:30~19:30이며, 재료 소진 시 조기 마감될 수 있습니다." },
            
            { id: 4, text: "제주시청 근처 '제주 고기국수'는 30년 전통의 고기국수 전문점입니다. 돔베고기와 고기국수가 대표메뉴이며, 고기국수 한 그릇이 8천원입니다. 진한 돼지고기 육수에 중면을 사용하여 깊은 맛을 자랑하며, 특제 육수는 매일 아침 새로 만듭니다. 고기국수와 함께 주문하는 돔베고기(중 2만원)는 기본 김치와 마늘, 양파 등 쌈 채소가 함께 제공됩니다. 영업시간은 9:00~21:00이며, 매주 일요일 휴무입니다. 주차장이 있어 편리하며, 점심시간에는 대기가 있을 수 있습니다." },
            
            { id: 5, text: "서귀포 올레시장 내 '올레 회국수'는 제주 현지인이 추천하는 회국수 맛집입니다. 신선한 생선회가 듬뿍 들어간 회국수가 인기메뉴이며, 한 그릇에 만원입니다. 제주 근해에서 잡은 신선한 생선으로 회를 떠서 사용하며, 특제 육수와 함께 제공됩니다. 기본 반찬으로 제주식 김치와 미역초무침이 제공되며, 공기밥(1천원)을 추가하면 식사가 더욱 알차집니다. 영업시간은 10:30~20:00이며, 매주 수요일 휴무입니다. 올레시장 내부에 위치해 있어 주차는 시장 주차장을 이용하면 됩니다." },
            
            { id: 6, text: "제주시 이도동의 '제주전통순대'는 40년 전통의 순대국밥 전문점입니다. 제주식 순대와 순대국밥이 대표메뉴이며, 순대국밥은 7천원입니다. 제주 흑돼지로 만든 순대와 진한 국물이 특징이며, 도민들의 아침식사 메뉴로도 인기가 많습니다. 순대만 따로 포장도 가능하며(1인분 1만원), 특별한 양념장과 함께 제공됩니다." },
            
            { id: 7, text: "서귀포시 색달동의 '제주 흑돼지명가'는 20년 경력의 셰프가 운영하는 흑돼지 전문점입니다. 흑돼지 샤브샤브와 흑돼지 돈까스가 대표메뉴이며, 흑돼지 샤브샤브는 2인 기준 5만원입니다. 100% 제주산 흑돼지만을 사용하며, 특제 육수와 소스가 맛의 비결입니다." },
            
            { id: 8, text: "제주시 노형동의 '제주 생선구이'는 제주 특산물인 각종 생선요리를 전문으로 하는 맛집입니다. 옥돔구이와 한치구이가 대표메뉴이며, 옥돔구이 한 마리가 2만원입니다. 제주 근해에서 잡은 신선한 생선만을 사용하며, 제철 생선으로 만드는 구이정식(2.5만원)이 특히 인기가 많습니다." }
        ];

        const url = `https://open-api.jejucodingcamp.workers.dev/`;
        const data = [];
        data.push({
            "role": "system",
            "content": `당신은 제주도 맛집 전문가입니다. 다음 정보를 바탕으로 질문에 답변해주세요:

            ${documents.map(d => d.text).join('\n\n')}

            답변 작성 지침:
            1. 항상 정중하고 친절한 어투를 사용하세요
            2. 맛집 정보를 추천할 때는 위치, 대표메뉴, 가격, 영업시간, 특징 순으로 설명하세요
            3. 가능한 한 상세하게 설명하되, 제공된 정보에 없는 내용은 답변하지 마세요
            4. 여러 옵션을 제시할 때는 번호를 매겨 구분하세요
            5. 답변 끝에는 추가 질문을 유도하는 문구를 넣으세요`
        });

        function handleUserInput() {
            const query = userInput.value.trim();
            if (!query) return;

            addMessage(query, true);
            
            // 로딩 메시지 추가
            const loadingMessageId = addLoadingMessage();
            
            // 사용자 질문 추가
            data.push({
                "role": "user",
                "content": query
            });

            // ChatGPT API 호출
            fetch(url, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data),
                redirect: 'follow'
            })
            .then(res => res.json())
            .then(res => {
                // 로딩 메시지 제거
                removeLoadingMessage(loadingMessageId);
                
                const answer = res.choices[0].message;
                data.push(answer); // 답변 저장
                addMessage(answer.content);
            })
            .catch(error => {
                // 로딩 메시지 제거
                removeLoadingMessage(loadingMessageId);
                
                console.error('Error:', error);
                addMessage("죄송합니다. 답변을 생성하는 중에 문제가 발생했습니다. 잠시 후 다시 시도해주세요.");
            });
            
            userInput.value = '';
        }

        const chatBox = document.getElementById('chat-box');
        const userInput = document.getElementById('user-input');
        const sendButton = document.getElementById('send-button');

        function addMessage(message, isUser = false) {
            const messageGroup = document.createElement('div');
            messageGroup.className = 'message-group';
            
            const messageDiv = document.createElement('div');
            messageDiv.className = isUser ? 'user-message' : 'assistant-message';
            
            const avatar = document.createElement('div');
            avatar.className = `message-avatar ${isUser ? 'user-avatar' : 'assistant-avatar'}`;
            avatar.textContent = isUser ? '나' : 'J';
            
            const content = document.createElement('div');
            content.className = 'message-content';
            content.innerHTML = message.replace(/\n/g, '<br>');
            
            messageDiv.appendChild(avatar);
            messageDiv.appendChild(content);
            messageGroup.appendChild(messageDiv);
            chatBox.appendChild(messageGroup);
            chatBox.scrollTop = chatBox.scrollHeight;
        }

        function addLoadingMessage() {
            const messageGroup = document.createElement('div');
            messageGroup.className = 'message-group';
            
            const messageDiv = document.createElement('div');
            messageDiv.className = 'assistant-message';
            
            const avatar = document.createElement('div');
            avatar.className = 'message-avatar assistant-avatar';
            avatar.textContent = 'J';
            
            const content = document.createElement('div');
            content.className = 'message-content loading-message';
            content.innerHTML = `
                <span>답변 생성 중</span>
                <div class="loading-dots">
                    <div class="loading-dot"></div>
                    <div class="loading-dot"></div>
                    <div class="loading-dot"></div>
                </div>
            `;
            
            messageDiv.appendChild(avatar);
            messageDiv.appendChild(content);
            messageGroup.appendChild(messageDiv);
            chatBox.appendChild(messageGroup);
            chatBox.scrollTop = chatBox.scrollHeight;
            
            return messageGroup;
        }

        function removeLoadingMessage(messageGroup) {
            if (messageGroup && messageGroup.parentNode) {
                messageGroup.parentNode.removeChild(messageGroup);
            }
        }

        sendButton.addEventListener('click', handleUserInput);
        userInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                handleUserInput();
            }
        });

        // 초기 메시지
        addMessage(`안녕하세요! 제주도 맛집 추천 챗봇입니다. 제주도의 맛있는 식당들을 상세히 안내해드립니다.

다음과 같은 정보를 물어보실 수 있습니다:
1. 지역별 맛집 (제주시, 서귀포, 중문, 성산 등)
2. 음식 종류별 맛집
   - 흑돼지 전문점
   - 갈치요리 전문점
   - 해녀식당/해산물
   - 고기국수/회국수
   - 순대국밥
   - 생선구이
3. 상세 정보
   - 가격대
   - 영업시간
   - 주차 가능 여부
   - 예약 필요 여부
   - 단체 가능 여부

무엇을 도와드릴까요? 맛있는 제주 맛집을 찾아드리겠습니다!`);
    </script>
</body>
</html>