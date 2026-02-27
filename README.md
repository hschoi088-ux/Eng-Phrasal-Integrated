<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스피드 영어 퀴즈 (업그레이드)</title>
    <style>
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif;
            background-color: #f0f4f8;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.08);
            max-width: 650px;
            width: 90%;
            text-align: center;
        }
        h1 { margin-bottom: 20px; color: #1e293b; font-size: 24px; }
        .btn-group { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
        button {
            padding: 12px 20px;
            border: none;
            border-radius: 10px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s ease;
        }
        .btn-mode { background-color: #3b82f6; color: white; }
        .btn-mode:hover { background-color: #2563eb; transform: translateY(-2px); }
        .hidden { display: none !important; }
        #quiz-area { display: flex; flex-direction: column; gap: 20px; }
        #question-counter { font-size: 15px; color: #64748b; font-weight: bold; }
        
        .question-box { 
            font-size: 22px; font-weight: bold; word-break: keep-all; line-height: 1.5; 
            background: #f8fafc; padding: 30px 20px; border-radius: 12px;
            border: 2px dashed #cbd5e1; cursor: pointer; transition: background 0.2s;
        }
        .question-box:hover { background: #e2e8f0; }
        .question-box::after { content: "\n(클릭하여 정답 확인)"; font-size: 13px; color: #94a3b8; display: block; margin-top: 10px; font-weight: normal; }

        .answer-box { 
            background: #ecfdf5; padding: 25px 20px; border-radius: 12px;
            border: 1px solid #a7f3d0; text-align: left;
        }
        .en-text { font-size: 22px; color: #059669; font-weight: bold; margin-bottom: 15px; line-height: 1.4; word-break: keep-all;}
        .meta-info { font-size: 14px; color: #475569; margin-bottom: 5px; background: #fff; display: inline-block; padding: 4px 10px; border-radius: 20px; border: 1px solid #e2e8f0; }
        .meaning-info { font-size: 15px; color: #b45309; margin-bottom: 15px; background: #fef3c7; padding: 8px 12px; border-radius: 8px; font-weight: 500;}
        
        .controls { display: flex; justify-content: space-between; align-items: center; margin-top: 10px; }
        .btn-tts { background-color: #8b5cf6; color: white; border-radius: 50px; padding: 8px 16px; font-size: 14px; display: flex; align-items: center; gap: 5px; }
        .btn-tts:hover { background-color: #7c3aed; }
        .btn-next { background-color: #10b981; color: white; padding: 12px 30px; }
        .btn-next:hover { background-color: #059669; }
        #btn-restart { background-color: #ef4444; color: white; width: 100%; padding: 15px; margin-top: 20px;}
        #btn-restart:hover { background-color: #dc2626; }
    </style>
</head>
<body>

<div class="container">
    <h1 id="main-title">🚀 스피드 영어 퀴즈</h1>
    
    <div id="mode-selection">
        <p style="color: #64748b; margin-bottom: 20px;">학습할 모드를 선택해 주세요. (랜덤 7문제)</p>
        <div class="btn-group">
            <button class="btn-mode" onclick="startQuiz('phrasal-easy')">🟢 구동사 순한맛</button>
            <button class="btn-mode" onclick="startQuiz('phrasal-hard')" style="background-color: #ef4444;">🔴 구동사 매운맛</button>
            <button class="btn-mode" onclick="startQuiz('conv-easy')" style="background-color: #10b981;">💬 영어회화 순한맛</button>
            <button class="btn-mode" onclick="startQuiz('conv-hard')" style="background-color: #f59e0b;">🔥 영어회화 매운맛</button>
        </div>
    </div>

    <div id="quiz-area" class="hidden">
        <div id="question-counter">문제 1 / 7</div>
        
        <div class="question-box" id="ko-box" onclick="showAnswer()">
            <span id="ko-text">한국어 문장이 여기에 표시됩니다.</span>
        </div>

        <div class="answer-box hidden" id="answer-section">
            <div class="meta-info" id="source-info">출처: Day001 교재1</div>
            <div class="en-text" id="en-text">English Answer</div>
            <div class="meaning-info hidden" id="meaning-info">의미: </div>
            
            <div class="controls">
                <button class="btn-tts" onclick="playTTS()">🔊 문장 듣기</button>
                <button class="btn-next" id="btn-next" onclick="nextQuestion()">다음 문제 ➡</button>
            </div>
        </div>

        <button id="btn-restart" class="hidden" onclick="resetQuiz()">처음으로 돌아가기</button>
    </div>
</div>

<script>
    // 구동사 의미 사전 (매핑용)
    const meanings = {
        "add up_1": "add up: 1. (수 등을) 하나하나 더하다",
        "add up_2": "add up: 2. (점진적으로 쌓여) 합계가 결국 ~가 되다",
        "add up_3": "add up: 3. (주로 부정문에서) 앞뒤가 맞다, 논리적으로 말이 되다",
        "blow away_1": "blow away: 1. (바람 등이 사물을) 멀리 날려 보내다",
        "blow away_2": "blow away: 2. (사람을) 대단히 놀라게 하거나 깊은 감명을 주다",
        "break down_1": "break down: 1. (기계나 차량이) 고장이 나다",
        "break down_2": "break down: 2. (협상이나 시스템 등이) 결렬되다, 실패로 끝나다",
        "break down_3": "break down: 3. (슬픔 등을 못 참고) 울며 무너지다, 자제력을 잃다",
        "break down_4": "break down: 4. (이해를 돕기 위해) 세부 사항을 조목조목 나누어 설명하다",
        "break up_1": "break up: 1. (연인 등이 관계를 끝내고) 헤어지다",
        "break up_2": "break up: 2. (여러 조각이나 작은 그룹으로) 분해하다, 분리하다",
        "break up_3": "break up: 3. (전화 신호 등이) 잡음과 함께 뚝뚝 끊기다",
        "brush up on_1": "brush up on: 1. (오래전 배운 지식이나 기술을) 다시 복습하다, 다듬다"
    };

    // 1. 구동사 순한맛 (13개)
    const phrasalEasy = [
        { ko: "자, 이 숫자들을 더해 보자.", en: "Let’s add up these numbers now.", source: "Day 001 순한맛", meaning: meanings["add up_1"] },
        { ko: "별것 아니게 보일 수 있어도, 하루 10분의 연습도 쌓이면 정말 큽니다.", en: "It might not seem like much, but 10 minutes of practice every day really adds up.", source: "Day 001 순한맛", meaning: meanings["add up_2"] },
        { ko: "뭔가 앞뒤가 안 맞잖아.", en: "Something doesn’t add up.", source: "Day 001 순한맛", meaning: meanings["add up_3"] },
        { ko: "모자가 강풍에 날아가 버렸다.", en: "My hat blew away in the strong wind.", source: "Day 002 순한맛", meaning: meanings["blow away_1"] },
        { ko: "정말 놀랐어요!", en: "You really blew me away!", source: "Day 002 순한맛", meaning: meanings["blow away_2"] },
        { ko: "제 차가 고속 도로에서 고장이 났습니다.", en: "My car broke down on the highway.", source: "Day 003 순한맛", meaning: meanings["break down_1"] },
        { ko: "근무시간 관련 협상이 결렬된 것은 불가피했습니다.", en: "It was inevitable that the negotiation over working hours broke down.", source: "Day 003 순한맛", meaning: meanings["break down_2"] },
        { ko: "의사 선생님이 제가 다시는 프로 선수로 뛸 수 없다고 했을 때 저는 무너졌습니다.", en: "When the doctor said I could never play professionally again, I broke down.", source: "Day 003 순한맛", meaning: meanings["break down_3"] },
        { ko: "스케줄을 세부적으로 말씀드릴게요.", en: "Let me break down the schedule.", source: "Day 003 순한맛", meaning: meanings["break down_4"] },
        { ko: "사실 Susie가 헤어지자고 해서 헤어진 거야.", en: "She broke up with me, actually.", source: "Day 004 순한맛", meaning: meanings["break up_1"] },
        { ko: "3명씩 조를 나누어라.", en: "I need you to break up into groups of three.", source: "Day 004 순한맛", meaning: meanings["break up_2"] },
        { ko: "네 말이 끊겨서 들려.", en: "You are breaking up.", source: "Day 004 순한맛", meaning: meanings["break up_3"] },
        { ko: "지난 학기에 배운 내용을 다시 한번 복습해 보겠습니다.", en: "I’d like us to brush up on what we learned last semester.", source: "Day 005 순한맛", meaning: meanings["brush up on_1"] }
    ];

    // 2. 구동사 매운맛 (전체 데이터 + 의미 매핑)
    const phrasalHard = [
        {ko: "별것 아니게 보일 수 있어도, 하루 10분의 연습도 쌓이면 정말 큽니다.", en: "It might not seem like much, but 10 minutes of practice every day really adds up.", source: "Day001 교재1", meaning: meanings["add up_2"]},
        {ko: "어제 집에 있었다고 했는데, 내 친구가 당신을 술집에서 봤다고 했어. 뭔가 앞뒤가 안 맞잖아.", en: "You told me you were at home, but my friend mentioned seeing you at a bar. Something doesn’t add up.", source: "Day001 교재1", meaning: meanings["add up_3"]},
        {ko: "자, 이 숫자들을 더해 보자. 5 더하기 3은 뭘까?", en: "Let’s add up these numbers now. What’s five plus three?", source: "Day001 교재1", meaning: meanings["add up_1"]},
        {ko: "월 10만 원도 쌓이면 4년 후에 거의 5백만 원이 된다.", en: "Just 100,000 won a month will add up to almost 5 million won in four years.", source: "Day001 교재1", meaning: meanings["add up_2"]},
        {ko: "그의 이야기에는 앞뒤가 맞지 않는 것이 있어요. 그가 거짓말을 하고 있는 게 틀림없어요.", en: "There’s something about his story that doesn’t add up. He must be telling a lie.", source: "Day001 교재1", meaning: meanings["add up_3"]},
        {ko: "한 달에 커피값으로 백 달러 정도를 지출해. 나의 길티 플레저거든.", en: "I spend around a hundred bucks a month on coffee. It’s my guilty pleasure.", source: "Day001 교재2", meaning: meanings["add up_2"]}, //문맥상 이어지는 내용
        {ko: "한 달에 백 달러도 5년이면 6천 달러야. 집에서 만들어 먹는 게 어때?", en: "A hundred bucks a month will add up to $6,000 in five years. Why don’t you make coffee at home?", source: "Day001 교재2", meaning: meanings["add up_2"]},
        {ko: "지난 분기 매출 수치를 합해서 목표치와 비교해 주시겠어요?", en: "Can you add up the sales figures from last quarter and compare them to our targets?", source: "Day001 교재2", meaning: meanings["add up_1"]},
        {ko: "물론입니다. 계산해서 오늘 오후까지 세부 보고서를 준비해 두겠습니다.", en: "Sure thing. I’ll do the math and have a detailed report ready by this afternoon.", source: "Day001 교재2", meaning: meanings["add up_1"]},
        {ko: "Brian은 너무 완벽해. 우리 다음 달에 오프라인에서 만날 거야.", en: "Brian is so perfect. We’re going to meet offline next month.", source: "Day001 교재2", meaning: meanings["add up_3"]},
        {ko: "그러니까 의사이자 파일럿이고, 외모는 모델 같은데, 아직 사진을 안 보여 줬다? 이건 말이 안 돼.", en: "So he’s a doctor, a pilot, and he looks like a model, yet he hasn’t shown you a photo? This doesn’t add up, Gina.", source: "Day001 교재2", meaning: meanings["add up_3"]},
        {ko: "이게 쌓이면 정말 큽니다.", en: "it really adds up.", source: "Day001 교재3", meaning: meanings["add up_2"]},
        {ko: "어젯밤에 눈이 많이 와서 미끄러우면 어쩌나 했지만, 바람이 워낙 강해서 눈이 다 날아가 버렸더군.", en: "I was worried that sidewalks would be slippery, but the wind was so strong that it blew the snow away.", source: "Day002 교재1", meaning: meanings["blow away_1"]},
        {ko: "그냥 압축공기를 이용해서 먼지를 날려 버린답니다.", en: "I just use compressed air to blow the dust away.", source: "Day002 교재1", meaning: meanings["blow away_1"]},
        {ko: "조심해! 자칫 날아갈 수도 있어!", en: "Be careful! You might get blown away!", source: "Day002 교재1", meaning: meanings["blow away_1"]},
        {ko: "이번에 새로 나온 태블릿을 처음 봤을 때 매우 인상적이었어.", en: "When I first saw their new tablet, I was blown away.", source: "Day002 교재1", meaning: meanings["blow away_2"]},
        {ko: "우와, 프레젠테이션 정말 유익했어요. 정말 놀랐어요!", en: "Wow, your presentation was so informative. You really blew me away!", source: "Day002 교재1", meaning: meanings["blow away_2"]},
        {ko: "바람이 많이 부는 날씨는 불쾌하지만, 적어도 미세 먼지를 날려 버리긴 하지.", en: "Windy weather is unpleasant, but at least it blows away all the microdust.", source: "Day002 교재1", meaning: meanings["blow away_1"]},
        {ko: "모자가 강풍에 날아가 버렸다.", en: "My hat blew away in the strong wind.", source: "Day002 교재1", meaning: meanings["blow away_1"]},
        {ko: "제 동생이 영어를 너무 잘해서 정말 놀랐어요.", en: "I was really blown away by his English.", source: "Day002 교재1", meaning: meanings["blow away_2"]},
        {ko: "바람이 너무 세서 자동차도 날아갔나 보더라고.", en: "Apparently, the wind was so strong that it even blew away cars.", source: "Day002 교재2", meaning: meanings["blow away_1"]},
        {ko: "주연 배우의 연기가 정말 인상적이더라고.", en: "I was absolutely blown away by the main actor’s performance.", source: "Day002 교재2", meaning: meanings["blow away_2"]},
        {ko: "김종국이 외국인들과 대화하는 거 봤는데 정말 대단하더라.", en: "I was completely blown away when I saw him chatting with foreigners.", source: "Day002 교재2", meaning: meanings["blow away_2"]},
        {ko: "메뉴 볼 때마다 깜짝 놀라. 만 오천 원 이하가 없다니까.", en: "I’m blown away every time I look at a menu. You can’t eat for less than 15,000 won these days.", source: "Day002 교재3", meaning: meanings["blow away_2"]},
        {ko: "몇 년을 매일 썼더니 컴퓨터가 결국 고장이 났다.", en: "The computer finally broke down after using it daily for years.", source: "Day003 교재1", meaning: meanings["break down_1"]},
        {ko: "근무시간 관련 협상이 결렬된 것은 불가피했습니다.", en: "It was inevitable that the negotiation over working hours broke down.", source: "Day003 교재1", meaning: meanings["break down_2"]},
        {ko: "의사 선생님이 제가 다시는 프로 선수로 뛸 수 없다고 했을 때 저는 무너졌습니다.", en: "When the doctor said I could never play professionally again, I broke down.", source: "Day003 교재1", meaning: meanings["break down_3"]},
        {ko: "스케줄을 세부적으로 말씀드릴게요.", en: "Let me break down the schedule.", source: "Day003 교재1", meaning: meanings["break down_4"]},
        {ko: "수치가 잘 이해되지 않네요. 미안한데 다시 한번 자세히 설명해 주시겠어요?", en: "Would you mind going back and breaking those down?", source: "Day003 교재1", meaning: meanings["break down_4"]},
        {ko: "담배꽁초가 분해되는 데 18개월에서 10년이 걸리는 거 알았어?", en: "Did you know that cigarette butts take between 18 months and 10 years to break down?", source: "Day003 교재1", meaning: meanings["break down_2"]}, //분해되다
        {ko: "제 차가 고속 도로에서 고장이 났습니다.", en: "My car broke down on the highway.", source: "Day003 교재1", meaning: meanings["break down_1"]},
        {ko: "항상 돈 이야기가 나오면 이런 대화가 깨집니다.", en: "but those talks always break down once money comes up.", source: "Day003 교재1", meaning: meanings["break down_2"]},
        {ko: "회사에서 계속 압박에 시달린 후에 결국 감정적으로 무너졌고 사무실에서 울었어요.", en: "After weeks of constant pressure at work, I finally broke down and cried in my office.", source: "Day003 교재1", meaning: meanings["break down_3"]},
        {ko: "문장을 좀 더 잘 이해하기 위해서, 여러 단위로 나눠서 표현 하나하나를 살펴봤습니다.", en: "In order to understand the sentence better, I broke it down into parts and looked at the phrases one by one.", source: "Day003 교재1", meaning: meanings["break down_4"]},
        {ko: "동물성 단백질은 분해되는 데 더 많은 에너지를 필요로 한다.", en: "Protein from meat requires more energy for your body to break down.", source: "Day003 교재1", meaning: meanings["break down_2"]},
        {ko: "사용하던 컴퓨터가 결국 고장이 났거든요.", en: "My old computer finally broke down.", source: "Day003 교재2", meaning: meanings["break down_1"]},
        {ko: "정신적으로 완전히 무너져서 아무 말도 안 나왔어.", en: "I just broke down completely and couldn’t even get words out.", source: "Day003 교재2", meaning: meanings["break down_3"]},
        {ko: "아파트 청약 제도를 잘 모르겠습니다. 자세히 좀 설명해 주시겠어요?", en: "I still don’t understand how apartment lotteries work. Could you break it down for me?", source: "Day003 교재2", meaning: meanings["break down_4"]},
        {ko: "버려진 후에 분해되는 데 정말 오래 걸려.", en: "They take forever to break down once they’re thrown away.", source: "Day003 교재3", meaning: meanings["break down_2"]},
        {ko: "너 Susie랑 헤어졌다는 게 사실이야?", en: "Is it true you broke up with Susie?", source: "Day004 교재1", meaning: meanings["break up_1"]},
        {ko: "사실 Susie가 헤어지자고 해서 헤어진 거야.", en: "She broke up with me, actually.", source: "Day004 교재1", meaning: meanings["break up_1"]},
        {ko: "구름이 서서히 걷히고 해가 더 밝아졌다.", en: "The clouds gradually broke up and the sun got brighter.", source: "Day004 교재1", meaning: meanings["break up_2"]},
        {ko: "닭고기를 한 입 크기로 찢은 다음, 샐러드에 넣어 섞으세요.", en: "First, use two forks to break up the chicken into bite-size pieces.", source: "Day004 교재1", meaning: meanings["break up_2"]},
        {ko: "여기 (지하라서) 신호가 끊겨.", en: "My signal is breaking up down here.", source: "Day004 교재1", meaning: meanings["break up_3"]},
        {ko: "사람들은 매일 같이 헤어지잖아. 너무 힘들게 받아들이지 마!", en: "People break up every day. Don’t take it so hard!", source: "Day004 교재1", meaning: meanings["break up_1"]},
        {ko: "자, 얘들아. 3명씩 조를 나누어라.", en: "OK, class. I need you to break up into groups of three.", source: "Day004 교재1", meaning: meanings["break up_2"]},
        {ko: "오노 요코 때문에 비틀즈가 해체됐다고 한다.", en: "My dad says Yoko Ono broke up The Beatles.", source: "Day004 교재1", meaning: meanings["break up_1"]}, //해체
        {ko: "저는 하루 일과를 다양한 종류의 업무로 쪼갭니다.", en: "I like to break up my day with various kinds of tasks.", source: "Day004 교재1", meaning: meanings["break up_2"]},
        {ko: "네 말이 끊겨서 들려.", en: "You are breaking up.", source: "Day004 교재1", meaning: meanings["break up_3"]},
        {ko: "아무도 안 볼 텐데. 좀 더 짧은 클립으로 쪼개야 해.", en: "No one is gonna watch that. You need to break it up into smaller clips.", source: "Day004 교재2", meaning: meanings["break up_2"]},
        {ko: "잠시만, 잘 안 들려. 신호가 계속 끊기네.", en: "Hold on, I can’t hear you. Your signal keeps breaking up.", source: "Day004 교재2", meaning: meanings["break up_3"]},
        {ko: "그러니까 밴드가 해산하는 건 미나 잘못이야.", en: "so it’s her fault the band is breaking up.", source: "Day004 교재2", meaning: meanings["break up_1"]},
        {ko: "하지만 급기야 둘 간의 관계가 흔들렸고 최근에 헤어지게 되었다.", en: "However, their relationship eventually got rocky, and they recently broke up.", source: "Day004 교재3", meaning: meanings["break up_1"]},
        {ko: "“민호랑 헤어진 마당에 서울에 계속 있어야 할 이유가 없어.”", en: "She told me, “Now that Minho and I broke up, I don’t have any reason to stay in Seoul.”", source: "Day004 교재3", meaning: meanings["break up_1"]},
        {ko: "다음 달에 파리로 여행 가는데, 프랑스어 복습 좀 해야겠어.", en: "I am travelling to Paris next month, so I think I need to brush up on my French.", source: "Day 5 교재1", meaning: meanings["brush up on_1"]},
        {ko: "지난 학기에 배운 내용을 다시 한번 복습해 보겠습니다.", en: "I’d like us to brush up on what we learned last semester.", source: "Day 5 교재1", meaning: meanings["brush up on_1"]},
        {ko: "빵 굽는 것을 연습하려면 요리책 읽어 봐야겠다.", en: "I’m going to read a cookbook so that I can brush up on my baking skills.", source: "Day 5 교재1", meaning: meanings["brush up on_1"]},
        {ko: "잊어버릴 만하면 (이 책에 담긴) 구동사를 복습하셔야 실제 상황에서 자신 있게 쓸 수 있습니다.", en: "You should brush up on these phrasal verbs from time to time in order to feel confident using them in real-life situations.", source: "Day 5 교재1", meaning: meanings["brush up on_1"]},
        {ko: "한동안 안 치던 기타도 연습하고 있어요.", en: "I’ve been brushing up on my guitar playing.", source: "Day 5 교재1", meaning: meanings["brush up on_1"]},
        {ko: "가기 전에 한국 역사 복습 좀 해야겠어.", en: "I want to brush up on my Korean history before we go.", source: "Day 5 교재2", meaning: meanings["brush up on_1"]},
        {ko: "이번 주는 집에서 거울 보고 연습하면서 발표 스킬을 좀 가다듬고 있어요.", en: "I’ve been brushing up on my presentation skills this week by practicing in the mirror at home.", source: "Day 5 교재2", meaning: meanings["brush up on_1"]},
        {ko: "그래서 요리 연습을 좀 해야 해요.", en: "I need to brush up on my cooking skills.", source: "Day 5 교재2", meaning: meanings["brush up on_1"]},
        {ko: "작업 멘트를 연습해 보는 것이 내가 생각할 수 있는 전부였다.", en: "All I could think to do was brush up on some pickup lines.", source: "Day 5 교재3", meaning: meanings["brush up on_1"]}
    ];

    // 3. 영어회화 전체 데이터 (전처리 용)
    const rawConvData = [
        { source: "Day001 교재1", ko: "저는 재택근무 체질이 아니에요. 늘 딴짓하게 되거든요", en: "Working from home isn’t for me. I always get distracted." },
        { source: "Day001 교재1", ko: "소개팅은 저랑 안 맞아요.", en: "Going on blind dates isn’t for me." },
        { source: "Day001 교재1", ko: "노트북은 저랑 좀 안 맞아요. 키보드가 뭔가 엄청 불편하거든요.", en: "Laptops aren’t really for me. Something about the keyboards is super uncomfortable." },
        { source: "Day001 교재1", ko: "전기차는 좀 별로예요. 충전소는 요즘 늘었지만, 여전히 엄청 귀찮게 느껴져요.", en: "Electric cars aren’t for me. We have more charging stations around now, but it still feels like too much of a hassle." },
        { source: "Day001 교재1", ko: "그 사람 직업이 좋은 건 아는데, 그런 남자는 나는 별로야.", en: "I know he has a decent job, but guys like him aren’t really for me." },
        { source: "Day001 교재2", ko: "우리 나가서 맛난 회 먹을까? 내가 살 게.", en: "Why don’t we go out and get some nice sashimi? My treat!" },
        { source: "Day001 교재2", ko: "너무 고맙긴 한데. 난 회를 별로 안 좋아해. 식감이 적응이 안 돼.", en: "It’s kind of you to offer, but raw fish just isn’t for me. I can’t get used to the texture." },
        { source: "Day001 교재2", ko: "청취 연습을 위해 <기묘한 이야기>를 시청할 것을 추천합니다.", en: "I recommend watching Stranger Things to practice listening." },
        { source: "Day001 교재2", ko: "좋은 생각이긴 한데, 저는 미국 프로그램이 체질에 안 맞아요. 스토리에 재미가 안 붙어요.", en: "It’s a good idea, but American shows aren’t for me. I can’t really get into the stories." },
        { source: "Day001 교재2", ko: "애들하고 정말 잘 노는군요. 선생님 할 생각은 해 보셨나요?", en: "You’re really great around kids. Have you ever thought of being a teacher?" },
        { source: "Day001 교재2", ko: "아니요. 저는 가르치는 거랑 잘 안 맞아요. 애들이랑 노는 건 좋은데, 공부시키는 게 너무 힘들 듯해요", en: "No, no. Teaching isn’t really for me. I like to play with them but trying to make them study seems like hard work." },
        { source: "Day001 교재3", ko: "안녕, Greg. 내가 생일 선물로 받은 로잉 머신 기억하지? 혹시 관심 있어? 나랑은 별로 안 맞더라고", en: "Hey, Greg. Do you remember that rowing machine I got for my birthday? Are you interested in it? Turns out it’s not really for me." },
        { source: "Day001 교재4", ko: "좋아 보인다. 편안해 보이고. 학과장을 안 하는 게 너랑 맞는 거야", en: "You look good. Relaxed. Not being chair suits you." },
        { source: "Day001 교재4", ko: "혼자 일하는 건 나랑 안 맞는다는 걸 느꼈어. 사람들이랑 같이 있어야 진짜로 능률이 오르거든.", en: "I’ve found that working on my own doesn’t really suit me. I need to be around other people to really be productive." },
        { source: "Day001 교재4", ko: "있잖아. 재택근무는 내 체질이 아니야.", en: "You know what? Working from home doesn’t really work for me." },
        { source: "Day001 대표", ko: "재택근무는 저랑 안 맞아요.", en: "Working from home isn’t for me." },
        { source: "Day002 교재1", ko: "다음 에피소드는 어떤 내용일지 궁금해 미치겠어.", en: "I can’t wait to see what the next episode will bring." },
        { source: "Day002 교재1", ko: "아내가 제 선물을 개봉할 때 어떤 표정일지 궁금해 죽겠습니다.", en: "I can’t wait to see the look on my wife’s face when she opens my gift." },
        { source: "Day002 교재1", ko: "이 프로젝트가 빨리 끝났으면 좋겠어요. 너무 오래 걸립니다.", en: "I can’t wait to be done with this project. It’s taking forever." },
        { source: "Day002 교재1", ko: "여보, 저녁 식사가 너무 맛있는 냄새가 나네. 어서 먹고 싶어.", en: "That dinner smells delicious, honey. I can’t wait." },
        { source: "Day002 교재1", ko: "<베이비 드라이버>가 미국에서는 몇 달 전에 개봉했어. 이곳에서도 어서 개봉했으면 좋겠다.", en: "Baby Driver was released months ago in the United States. I can’t wait for it to come out here." },
        { source: "Day002 교재2", ko: "그 책 드디어 영화로 만들었다며?", en: "Did you hear they finally made that book into a movie?" },
        { source: "Day002 교재2", ko: "응! 어서 보고 싶어. 내가 제일 좋아하는 장면들이 다 포함되어 있기를.", en: "Yes! I can’t wait to see it. I hope they included all my favorite scenes." },
        { source: "Day002 교재2", ko: "프로젝트는 잘 되어 가나요? 한동안 매달려 있으신 것 같은데.", en: "How’s that project going? It seems like you’ve been working on it for a while." },
        { source: "Day002 교재2", ko: "네, 일주일 내내 이것을 하고 있습니다. 어서 끝내고 뭔가 다른 걸로 넘어가고 싶어요.", en: "Yeah, I’ve been working on it all week. I can’t wait to finish it and finally move on to something else." },
        { source: "Day002 교재2", ko: "네가 뜨개질할 수 있는 걸 몰랐네. 뭐 만들고 있니?", en: "I didn’t know you could knit. What are you making?" },
        { source: "Day002 교재2", ko: "여동생에게 줄 스카프를 만들고 있어. 내가 자기 주려고 이걸 만든 걸 알면 어떤 표정일까 궁금해 죽겠어.", en: "I’m making a scarf for my little sister. I can’t wait to see the look on her face when she realizes I made this for her." },
        { source: "Day002 교재3", ko: "신형 그랜저를 어서 보고 싶네요. 위장막 때문에 스파이 샷이 의미가 없어서 너무 속상하네요.", en: "I can’t wait to get a glimpse of the new Grandeur. Hyundai uses these car camouflage wraps. They make spy shots useless and it’s really frustrating." },
        { source: "Day002 교재4", ko: "(소개팅 상황에서) 그 여성분 어서 만나 보고 싶어.", en: "I’m really anxious to meet her." },
        { source: "Day002 교재4", ko: "어서 집에 가서 선물을 개봉해 보고 싶다.", en: "I’m anxious to get home to open my presents." },
        { source: "Day002 교재4", ko: "(이메일에서) 말씀 많이 들었습니다. 하루빨리 함께 일하고 싶습니다.", en: "I’ve heard a great deal about you. I look forward to working with you." },
        { source: "Day002 교재4", ko: "안 그래도 그 콘서트 너무 기대됐는데, 아이유도 나온다고 하니까 더 기대된다.", en: "I was looking forward to the concert, but now even more so, since I heard IU will be there." },
        { source: "Day002 대표", ko: "하루빨리 새 집으로 이사 가고 싶어요.", en: "I can’t wait to move into the new house." },
        { source: "Day003 교재1", ko: "제가 마지막 남은 피자 한 조각 먹어도 될까요?", en: "Do you mind if I finish off the last piece of pizza?" },
        { source: "Day003 교재1", ko: "미안한데, 오는 길에 커피 좀 사다 줄 수 있나요?", en: "Do you mind grabbing me some coffee on your way?" },
        { source: "Day003 교재1", ko: "제가 여유 시간이 겨우 5분 있어요. 짧게 해 주실 수 있을까요?", en: "I’ve got only five minutes to spare. Do you mind keeping it short?" },
        { source: "Day003 교재1", ko: "에어컨 좀 약하게 하면 안 될까요? 좀 추워서요.", en: "Do you mind turning down the air-conditioning? I feel a bit cold." },
        { source: "Day003 교재1", ko: "개인적인 질문 하나 해도 될까요?", en: "Do you mind if I ask you a personal question?" },
        { source: "Day003 교재2", ko: "죄송한데, 회의를 금요일로 옮겨도 될까요?", en: "Do you mind if we move the meeting to Friday?" },
        { source: "Day003 교재2", ko: "네, 괜찮습니다. 사실 저희에겐 금요일이 더 좋아요.", en: "Sure. Friday works better for us, actually." },
        { source: "Day003 교재2", ko: "죄송한데, 꼭대기 선반에 있는 저 시리얼 상자들 중 하나를 내려 줄 수 있을까요?", en: "Excuse me, do you mind grabbing me one of those cereal boxes on the top shelf?" },
        { source: "Day003 교재2", ko: "당연하죠. 얼마든지요", en: "Sure. Always happy to help!" },
        { source: "Day003 교재2", ko: "어디서 만나면 될까요? (비즈니스적)", en: "Where would you like to meet?" },
        { source: "Day003 교재2", ko: "제가 그쪽 사무실로 가도 상관없습니다.", en: "I don’t mind coming over to your office." },
        { source: "Day003 교재3", ko: "괜찮으시면 혹시 모르니까 이번에는 2시 30분에 시작해도 될는지요?", en: "If you don’t mind, could we start at 2:30 this time, just to be safe?" },
        { source: "Day003 교재4", ko: "제가 이번 주 금요일에 이사 나가요. 미안하지만 좀 도와주실 수 있을까요?", en: "I’m moving out this Friday. Would you mind giving me a hand?" },
        { source: "Day003 교재4", ko: "슬슬 배가 고프네요. 잠깐 나가서 간단히 뭐 좀 먹고 와도 될까요?", en: "I’m starting to get a bit hungry. Would you mind if I stepped out for a moment and grabbed a bite to eat?" },
        { source: "Day003 대표", ko: "죄송한데 조금 짧게 해 주시겠어요?", en: "Do you mind keeping it a bit short?" },
        { source: "Day004 교재1", ko: "그 여자분 키 엄청 커요.", en: "She is super tall." },
        { source: "Day004 교재1", ko: "그 사람이 무지 바쁘거나, 아니면 저에 대한 관심이 식고 있는 거겠죠.", en: "Either he has been super busy, or he is losing interest in me." },
        { source: "Day004 교재1", ko: "제가 요즘 이사 준비 때문에 엄청 바빴어요.", en: "I’ve been super busy with my upcoming move." },
        { source: "Day004 교재1", ko: "우와. 연세 있으신 분치고는 몸매가 너무 좋으시네요.", en: "Wow. You’re in super good shape for an old guy." },
        { source: "Day004 교재1", ko: "서울은 어디라도 다 너무 비싸. 근데 후암동은 상대적으로 저렴한 편이지", en: "All the neighborhoods in Seoul are super expensive, but Huam-dong is relatively cheap" },
        { source: "Day004 교재2", ko: "무슨 점심값이 만 원이 넘는 거야.", en: "I never thought I’d have to pay over 10,000 won for lunch." },
        { source: "Day004 교재2", ko: "그러게. 요새 물가가 너무너무 비싸.", en: "Yeah. Everything is getting super expensive." },
        { source: "Day004 교재2", ko: "오! 그럼 귤 한 박스 사다 줄래? 지금 제철이니 엄청 쌀 거야.", en: "Oh! How about a box of tangerines? They should be super cheap since they’re in-season." },
        { source: "Day004 교재2", ko: "11월 말치고는 너무 따뜻하다. 지금쯤이면 보통은 훨씬 더 추운데.", en: "It’s unusually warm for late November. It’s usually much colder by now." },
        { source: "Day004 교재2", ko: "맞아. 가을이 점점 짧아지고는 있는데 올해는 엄청 길다.", en: "Right. Autumn has been getting shorter, but this year, it’s been super long." },
        { source: "Day004 교재3", ko: "이번 침대 프레임 조립은 정말 쉽더군요. 조립하는 데 한 시간도 안 걸렸습니다.", en: "this bed frame was super easy to put together. It took me less than an hour." },
        { source: "Day004 교재4", ko: "그 삼겹살집은 맛은 괜찮은 편인데 가격이 상당히 비싸다.", en: "The pork belly place is pretty good, but it’s quite expensive." },
        { source: "Day004 교재4", ko: "당신은 영어를 상당히 잘하는군요.", en: "Your English is quite good." },
        { source: "Day004 대표", ko: "물가가 올라도 너무 올라요.", en: "Everything is getting super expensive." },
        { source: "Day005 교재1", ko: "중매업체에 등록해 보는 게 어때요?", en: "How do you feel about signing up for a matchmaking service?" },
        { source: "Day005 교재1", ko: "교회에 가 보는 게 어때요?", en: "How do you feel about going to church?" },
        { source: "Day005 교재1", ko: "등산 모임에 가입해 보는 게 어떨까요?", en: "How do you feel about joining a hiking club?" },
        { source: "Day005 교재1", ko: "성형 수술 하는 거 어떻게 생각하세요?", en: "How do you feel about plastic surgery?" },
        { source: "Day005 교재2", ko: "코치가 전 동료였는데, 그런 팀에 합류하는 기분이 어떠신가요?", en: "How do you feel about joining a team when the coach is your ex-teammate?" },
        { source: "Day005 교재2", ko: "저녁 먹고 우리 집에 가서 <컨저링> 볼까 하는데. 공포 영화 어때?", en: "After dinner, I was thinking we could go to my place and watch The Conjuring. How do you feel about horror movies?" },
        { source: "Day005 교재2", ko: "싫어, 공포 영화는 못 보겠어. 무서운 거 보는 게 뭐가 재미있다고.", en: "No, I can’t stand horror movies! Watching something scary isn’t my idea of fun." },
        { source: "Day005 교재3", ko: "Frank 팀장님 밑에서 일하니까 어떤가요? 이동 제안을 진지하게 고민해보기 전에 우선 당신의 경험을 듣고 싶습니다.", en: "How do you feel about working under Frank? I want to hear about your experience before I really consider his transfer offer." },
        { source: "Day005 교재4", ko: "모든 게 완전 비싸네.", en: "Everything is totally overpriced." },
        { source: "Day005 교재4", ko: "완전 깜박했어.", en: "It totally slipped my mind." },
        { source: "Day005 교재4", ko: "양복 입으니까 완전 딴 사람 같네.", en: "You look totally different in a suit." },
        { source: "Day005 교재4", ko: "완전 괜찮아.", en: "That’s totally fine." },
        { source: "Day005 교재4", ko: "괜히 고치느라 애쓰지 마. 완전히 고장이 났으니까.", en: "Don’t bother trying to fix it. It’s totally broken." },
        { source: "Day005 교재4", ko: "그 영화는 무조건 아이맥스로 봐야 해.", en: "You should totally go see the movie in IMAX." },
        { source: "Day005 교재4", ko: "난 초밥이 너무 땡겨.", en: "I could totally go for some sushi." }
    ];

    // 영어회화 분류 (순한맛: 대표 or 교재1 / 매운맛: 그 외)
    const convEasy = rawConvData.filter(item => item.source.includes('대표') || item.source.includes('교재1'));
    const convHard = rawConvData.filter(item => !(item.source.includes('대표') || item.source.includes('교재1')));

    let currentQuestions = [];
    let currentIndex = 0;
    let isPhrasalMode = false;

    // 배열 셔플 함수
    function shuffleArray(array) {
        let shuffled = [...array];
        for (let i = shuffled.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
        }
        return shuffled;
    }

    // 퀴즈 시작
    function startQuiz(mode) {
        document.getElementById('mode-selection').classList.add('hidden');
        document.getElementById('quiz-area').classList.remove('hidden');

        let dataPool = [];
        if (mode === 'phrasal-easy') { dataPool = phrasalEasy; isPhrasalMode = true; document.getElementById('main-title').innerText = "🟢 구동사 순한맛 퀴즈"; }
        else if (mode === 'phrasal-hard') { dataPool = phrasalHard; isPhrasalMode = true; document.getElementById('main-title').innerText = "🔴 구동사 매운맛 퀴즈"; }
        else if (mode === 'conv-easy') { dataPool = convEasy; isPhrasalMode = false; document.getElementById('main-title').innerText = "💬 영어회화 순한맛 퀴즈"; }
        else if (mode === 'conv-hard') { dataPool = convHard; isPhrasalMode = false; document.getElementById('main-title').innerText = "🔥 영어회화 매운맛 퀴즈"; }

        // 7문제 랜덤 추출
        currentQuestions = shuffleArray(dataPool).slice(0, 7);
        currentIndex = 0;
        
        loadQuestion();
    }

    // 문제 로드
    function loadQuestion() {
        document.getElementById('answer-section').classList.add('hidden');
        document.getElementById('btn-next').classList.add('hidden');
        document.getElementById('btn-restart').classList.add('hidden');
        document.getElementById('meaning-info').classList.add('hidden');
        
        // 박스 초기화 (클릭 유도)
        const koBox = document.getElementById('ko-box');
        koBox.style.cursor = 'pointer';
        koBox.style.pointerEvents = 'auto';

        const q = currentQuestions[currentIndex];
        document.getElementById('question-counter').innerText = `문제 ${currentIndex + 1} / 7`;
        document.getElementById('ko-text').innerText = q.ko;
        document.getElementById('en-text').innerText = q.en;
        document.getElementById('source-info').innerText = `출처: ${q.source}`;
        
        // 구동사 의미 처리
        if(isPhrasalMode && q.meaning) {
            document.getElementById('meaning-info').innerText = q.meaning;
        }
    }

    // 정답 확인 (박스 클릭 시)
    function showAnswer() {
        const koBox = document.getElementById('ko-box');
        koBox.style.cursor = 'default';
        koBox.style.pointerEvents = 'none';

        document.getElementById('answer-section').classList.remove('hidden');
        
        if(isPhrasalMode) {
            document.getElementById('meaning-info').classList.remove('hidden');
        }
        
        if (currentIndex < 6) {
            document.getElementById('btn-next').classList.remove('hidden');
        } else {
            document.getElementById('btn-restart').classList.remove('hidden');
        }
    }

    // TTS 기능
    function playTTS() {
        const textToSpeak = currentQuestions[currentIndex].en;
        if ('speechSynthesis' in window) {
            // 실행 중인 음성 취소
            window.speechSynthesis.cancel();
            const utterance = new SpeechSynthesisUtterance(textToSpeak);
            utterance.lang = 'en-US';
            utterance.rate = 0.9; // 약간 천천히 읽어줌
            window.speechSynthesis.speak(utterance);
        } else {
            alert('이 브라우저에서는 음성 듣기 기능을 지원하지 않습니다.');
        }
    }

    // 다음 문제로
    function nextQuestion() {
        currentIndex++;
        loadQuestion();
    }

    // 처음으로 돌아가기
    function resetQuiz() {
        document.getElementById('quiz-area').classList.add('hidden');
        document.getElementById('mode-selection').classList.remove('hidden');
        document.getElementById('main-title').innerText = "🚀 스피드 영어 퀴즈";
    }
</script>

</body>
</html>
