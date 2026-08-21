# 專案說明:拾光練習(互動英語/國語/數學練習網站)

這份文件是給「打開這個專案資料夾的 Claude Code」看的背景說明,目的是讓它不用重新摸索就能接手維護這個網站。這個 App 是使用者為了陪女兒(約 7-8 歲)一起學習而做的。**⚠️ 這份文件舊版曾誤把「酥酥老師」當成使用者本人的稱呼**——那其實只是網站草創期拿來當英文故事文章來源的網路文章署名,跟使用者無關,已經修正,之後請一律用「使用者」稱呼她。她本人**不是工程師**,溝通時請用白話文解釋做了什麼改動,不要只丟技術術語。

網站在 2026-08-20 從「互動英語練習」改名為「**拾光練習**」,因為內容已經擴充到英文/國語/數學三科,原本的名字只提到英文,已經不合身。改名出現在 `<title>`、頁尾文字、`HOME_HEADER.english.title`(原本是 `🎭 互動英語練習`,改成 `🎭 英文練習`,跟國語/數學的命名方式對齊)這幾個地方。

## 這是什麼

一個「單一 HTML 檔案」的互動學習網站,給小孩練習三科:

- **英文**:故事文章、Phonics(自然發音)、單字卡、文法、拼讀組合練習、闖關遊戲、還有「換我說說看」造句練習(有拼字校正)。
- **國語**:國字生字(康軒二上課文生字)、成語,目前只有故事劇場+闖關遊戲兩種練習方式。
- **數學**(2026-08-20 新增):目前只有九九乘法,重點是「理解」不是死背 —— 格子圖視覺化(看陣列理解交換律)、規律心法(找規律、拆解記憶技巧)、過關挑戰(混合直接背誦題、缺格找答案、生活應用題)。詳見下方「數學科(九九乘法)」那一節。

**唯一的檔案是 `index.html`**,所有 HTML/CSS/JavaScript 都寫在同一個檔案裡,沒有其他檔案、沒有 build 流程、沒有 npm/webpack 之類的東西。改完直接存檔、整個檔案就是成品。

## 檔案位置與部署流程

- 本機工作資料夾(這台電腦):`C:\Users\WanLing\Desktop\kidsDailyPractice\kidsDailyPractice\index.html`
- 這是一個 **git repo**,遠端是 GitHub:`https://github.com/lynnlee00/kidsDailyPractice`
- 網站用 **GitHub Pages** 架設,正式網址是:`https://lynnlee00.github.io/kidsDailyPractice/`
- Repo 是 **Public**(GitHub Pages 免費版必須是公開專案才能用)

### 更新流程(每次改完程式碼之後)

1. 確認改動只動 `index.html` 這一個檔案。
2. `git add index.html`
3. `git commit -m "說明這次改了什麼"`
4. `git push`
5. 等 1-2 分鐘讓 GitHub Pages 重新部署,再用 **強制重新整理**(Windows: Ctrl+F5)開網站確認,因為瀏覽器很容易吃到舊的快取版本。

使用者是用一個圖形化的 git 工具(畫面上按鈕文字是「提交/拉取/推送/獲取/分支/合併/貯藏/丟棄/標籤」)在操作,不是打指令。跟她說步驟時,講「按上面的推送按鈕」會比講 `git push` 更有幫助。

### ⚠️ 重要:不要嘗試用雲端沙盒環境直接 push

之前測試過,如果是在 Anthropic 雲端沙盒(Cowork 雲端模式)裡執行 `git push`,一定會被沙盒自己的 network proxy 擋下來,錯誤訊息類似:

```
remote: access denied by the git proxy: lynnlee00/kidsDailyPractice is not in this session's authorized repository set...
```

這跟 token 權限無關,是平台層級的限制,目前沒有辦法繞過。**push 這個動作只能在本機(這台電腦)上執行**,雲端 session 最多只能把新版 `index.html` 寫進本機資料夾,commit + push 一定要在本機做(不管是用圖形化 git 工具,還是在這裡用 Claude Code / 終端機執行 git 指令都可以,只要是「本機」執行就沒問題)。

### 版本號機制

網頁最下方 footer 有一個 `<span id="appVersion">` 版本號,例如 `v2026.08.20-1`。**每次有實質內容變動,請把這個版號往上加一碼**(例如 -1 → -2)。這是刻意加的,目的是讓使用者打開網站時,只要看這個小數字對不對,就能一眼判斷「網站是不是真的更新成功了」,不用靠感覺猜是不是快取問題。

## 程式架構

整個網站是「資料驅動」的:所有章節內容都放在一個叫 `CHAPTERS` 的大陣列裡,渲染畫面的函式都是根據 `category`/`subject`/`enabledTabs` 這些欄位去判斷要顯示什麼,而不是每個章節各寫一套畫面邏輯。**新增內容時,絕大部分只需要在 `CHAPTERS` 裡加資料,不需要動渲染邏輯。**

### 章節物件的關鍵欄位

- `id`:唯一識別碼,例如 `'character7'`、`'phonics3'`
- `subject`:`'english'` 或 `'mandarin'` — 決定首頁上方那個科目切換選單要不要顯示這個章節
- `category`:`'story'`(一般文章)/ `'phonics'`(自然發音)/ `'idiom'`(成語)/ `'character'`(國字生字)— 決定要用哪一種故事渲染方式(`renderStage` / `renderIdiomStage` / `renderCharacterStage`)
- `enabledTabs`:這個章節要顯示哪些分頁的陣列。沒有這個欄位的章節(目前所有英文一般文章/Phonics 章節)預設是全部 6 個分頁都顯示:`['story','vocab','grammar','blend','game','homework']`。國字生字、成語章節都明確設定成 `['story','game']`,只給故事劇場+闖關遊戲。
- `levels`:闖關遊戲的題目陣列,每一關對應一顆星星進度

### 分頁(Tab)自動隱藏機制(2026-08-20 新增)

分頁列有 7 顆按鈕:🏠章節選單 / 📖故事劇場(story) / 🎴單字卡(vocab) / 🪄文法魔法鏡(grammar) / 🧩組合練習(blend) / 🎮闖關遊戲(game) / ✏️換我說說看(homework)。

進入某個章節時,`enterChapter()` 會把不在該章節 `enabledTabs` 裡的按鈕設成 `disabled = true`。CSS 那邊有一條規則:

```css
nav.tabs button:disabled{
  display:none;
}
```

只要按鈕被設成 disabled,就會直接從畫面上消失,不是只變灰色。**這代表「隱藏」這個行為完全是靠 `disabled` 屬性驅動的 —— 之後如果想讓某個分頁顯示,絕對不要只改 CSS,要去確認 `enabledTabs` 陣列裡有沒有包含那個分頁名稱。**

`blend`(組合練習)分頁比較特別:它是不是要顯示,還要多看一個條件 `hasBlends = !!(ch.blends && ch.blends.length)`,只有章節資料裡真的有 `blends` 陣列(而且不是空的)才會顯示,即使 `enabledTabs` 裡有列 `'blend'` 也一樣。

還沒進入任何章節的首頁狀態,所有分頁按鈕預設是 disabled(HTML 原始碼裡就寫了 `disabled` 屬性),所以首頁只會看到「🏠 章節選單」一個按鈕,這是正常、預期的行為。

### TTS(語音朗讀)同音字校正機制

瀏覽器內建的語音合成(SpeechSynthesis)遇到中文多音字(破音字),常常會唸錯讀音(例如「吐」在「嘔吐」這個語境應該唸 4 聲 tù,但瀏覽器常常唸成 3 聲)。這個問題**沒辦法用修改顯示文字來解決**,因為顯示的字不能改(小孩要看到正確的國字),只能想辦法讓「語音引擎唸出來的內容」跟「畫面顯示的文字」分開處理。

解法是 `speakHighlight(el, text, opts)` 這個函式多加了一個可選參數 `speakText`:

- 畫面上的文字、逐字 highlight(跟著念的黃色標記)都還是用 `text`
- 但語音合成引擎實際念的內容,如果有給 `speakText` 就唸 `speakText`,沒給就照舊唸 `text`

用法是在資料裡加一個「同音字替換過」的版本,例如:

```js
{ reading:'ㄊㄨˋ', example:'弟弟坐車坐太久,吐了出來。', speakExample:'弟弟坐車坐太久,兔了出來。' }
```

`speakExample` 把「吐」換成同調同音的「兔」,唸出來聲音是對的,但畫面顯示的還是原本正確的「吐」字。

**⚠️ 極重要的限制:`speakText`/`speakExample`/`speakSentence` 這些替換版本的文字,長度必須跟原文字完全一樣(逐字對應),不能多字或少字。** 因為逐字 highlight 是靠字元索引位置去對應 `onboundary` 事件,如果替換後長度不一樣,highlight 的位置就會全部跑掉、跟語音對不上。只能做「一字換一字」的同音字替換,不能整句改寫。

目前用到這個機制的欄位:
- `polyphonic[].speakExample`(多音字卡片的例句語音)
- `words[].speakSentence`(單字例句語音)

如果之後又發現哪個字被瀏覽器唸錯,照同樣的模式加資料就好,不用改函式本身。

### TTS「唸出這一頁」導覽中斷機制(2026-08-21 修正,重要)

使用者截圖回報過一個 bug:點了「🔊 唸出這一頁」(`readAllBtn`,一次唸完一整頁好幾句話),還沒等它唸完就按「下一頁」,結果語音沒有停,還是繼續唸舊那頁的內容蓋過新頁面。

**根本原因**:`speakHighlightSequence()`(讓好幾句話接力唸完的函式)靠每一句話唸完的 `onend` 事件觸發下一句,但每句話的 `speakHighlight()` 內部還有一個「保險用的 `setTimeout`」(防止瀏覽器沒有正確觸發 `onend` 事件時卡住 UI)。使用者按「下一頁」時,雖然 `nextBtn` 的 click handler 會呼叫 `speechSynthesis.cancel()` 把「現在正在唸的那句話」停掉,但這個保險 `setTimeout` 早就已經排好在未來某個時間點觸發、完全不知道使用者已經按了下一頁,時間到了照樣觸發 `finish()`,而 `finish()` 裡面的 `onEnd` callback 會讓 `speakHighlightSequence` 的接力鏈繼續 `i++` 唸下一句——**於是舊頁面的語音接力鏈完全不受 `cancel()` 影響,繼續唸下去**。

**修法**:新增一個全域計數器 `activeSequenceId` 跟一個包裝函式 `cancelSpeech()`(=`activeSequenceId++` + `speechSynthesis.cancel()`)。`speakHighlightSequence()` 呼叫時會用 `const mySeq = ++activeSequenceId` 記住自己這一輪接力鏈的「身分證號碼」,每次要接著唸下一句之前,都會先檢查 `mySeq !== activeSequenceId`,如果不相等(代表中途有任何「導覽/離開」動作發生過,把全域計數器往上加了),就直接放棄、不再繼續唸。

**所有原本會呼叫 `speechSynthesis.cancel()` 的「導覽類」動作**(切換分頁、上一頁/下一頁——包含成語卡/生字卡/乘法格子圖/一般故事、交換排排看按鈕、拼讀組合練習的上一頁/下一頁、文法魔法鏡切換、`enterChapter()`、`goHome()`)**全部改成呼叫 `cancelSpeech()`**,而不是直接呼叫 `speechSynthesis.cancel()`——這樣才會讓 `activeSequenceId` 真的往上加,接力鏈才抓得到「使用者已經離開了」。

**之後如果要新增任何會讓使用者離開目前畫面/切換內容的按鈕或動作,只要牽涉到語音可能還在播放,一律呼叫 `cancelSpeech()`,不要直接寫 `speechSynthesis.cancel()`**——否則就會重現這個 bug(單一句子的語音會停,但「一次唸完整頁」的接力鏈不會停)。`speak()`/`speakHighlight()` 這兩個底層函式本身已經處理好了:`speak()` 每次呼叫都會自動 `cancelSpeech()`;`speakHighlight()` 只有在「不是被 `speakHighlightSequence` 呼叫」(也就是使用者直接點某個單一句子/單字的 🔊 按鈕)的情況下才會自動 `cancelSpeech()`,如果是接力鏈內部呼叫(帶了內部參數 `opts._seq`),就不會誤將自己的接力鏈判定為過期——這兩個函式不需要额外處理,只有「導覽類」的呼叫點需要注意。

**⚠️ 測試這類 TTS 相關 bug 時的環境地雷**:這個專案的測試環境(Playwright + headless Chromium)裡,`window.speechSynthesis = {...}`(整個物件重新賦值)會被瀏覽器靜默拒絕(這個屬性是不可重新賦值的),導致測試以為 mock 生效了,實際上 App 呼叫的還是瀏覽器原生的 SpeechSynthesis,常常會看到主控台跳出 `Failed to execute 'speak' on 'SpeechSynthesis': parameter 1 is not of type 'SpeechSynthesisUtterance'` 這種錯誤。**正確做法是不要整個物件重新賦值,而是直接改寫原生物件身上個別的方法**,例如:
```js
speechSynthesis.getVoices = () => [];
speechSynthesis.speak = (u) => { /* 記錄或什麼都不做 */ };
speechSynthesis.cancel = () => {};
window.SpeechSynthesisUtterance = function(text){ this.text = text; };
```
這樣才能真的攔截到 App 呼叫的語音函式,測試才有意義。

### 「換我說說看」拼字校正功能(2026-08-20 新增)

只有 `enabledTabs` 有包含 `'homework'` 的章節才有這個功能(目前是 unit11 跟 phonics1-5,國字/成語章節沒有這個分頁)。

小朋友在「I want / I need」的空格裡打完英文單字、按🔊之後:

1. `checkVocabSpelling(typed)` 會拿這個字去比對這一課 `vocab` 清單裡的每個單字,用 **Levenshtein 編輯距離**(`editDistanceOps`)算出「打的字」跟「清單裡每個字」差多少
2. 完全比對得上 → 狀態 `'correct'`,綠色✅
3. 差距很小(判定是打字失誤等級的差距)→ 狀態 `'close'`,黃色⚠️,用 `diffHighlightHtml` 把正確拼法裡「打錯的那幾個字母」用紅色底線標出來,方便小孩比對,同時附上這個字的中文意思。**語音會念修正後的正確拼法,不會照小孩打錯的唸。**
4. 完全對不上清單裡任何一個字 → 狀態 `'unknown'`,藍色💡,溫和提示她去單字卡裡找

相關函式都寫在 `editDistanceOps` / `diffHighlightHtml` / `checkVocabSpelling` / `renderHwFeedback` 這幾個,在 `#hwSpeakBtn` 的 click handler 裡串起來用。切換章節時 `initHomeworkUI()` 會把上一課殘留的提示訊息清空。

### 拼音顯示規則(容易搞混,要注意)

- **國字生字章節(`category:'character'`)**:`pinyin` 欄位跟多音字的 `reading` 欄位,全部都是**注音符號**(ㄅㄆㄇㄈ),不是羅馬拼音。這是 2026 這次改版特地全部換過的,使用者明確要求「不要羅馬拼音」。
- **成語章節(`category:'idiom'`)**:`pinyin` 欄位維持**漢語拼音**(羅馬拼音),格式是空格分隔多音節,例如 `'yi wu suo huo'`,這個**沒有**被轉換,是刻意保留的。
- 如果之後要加新的國字生字章節,新加的 `pinyin`/`reading` 欄位請直接寫注音符號,不要寫羅馬拼音,才會跟現有內容一致。

### 造詞題目的設計原則

國字生字章節裡的「造詞」題目,曾經被抓出一個設計缺陷:題目問「哪一個是『X』的正確造詞」,但選項本身就已經暗示答案是哪個字(例如題目配的插圖只跟其中一個選項有關),小孩不用真的認識這個字就能用刪去法猜對。

修正方式是把這種題目全部改成**克漏字**:直接借用這個詞語自己原本就有的例句,把詞語挖空,讓小孩從選項裡選出正確的詞語填進去(`before` + `after` + `options`,例句本身當題目情境,而不是用「哪個是正確造詞」這種後設問法)。**之後如果要新增造詞類題目,請照這個克漏字格式,不要用「哪一個是 X 的正確造詞」這種問法**,因為那種問法的選項設計很容易不小心洩題。

### 生字卡「更多造詞」欄位(`moreWords`,2026-08-21 新增)

使用者看了生字卡畫面(story 分頁)後要求:在「✏️ 造詞・造句」那個區塊下面,再加一排「純造詞、不用例句」的詞語,讓小孩多累積一些詞彙印象,不需要每個詞都配一整句例句。

做法是幫 `characters[]` 每個字加一個新的可選欄位 `moreWords: ['詞語1','詞語2',...]`(純字串陣列,**不是** `words[]` 那種 `{word, sentence}` 物件),`renderCharacterStage()` 在「造詞・造句」區塊下面、破音字區塊上面,多渲染一個「💡 更多造詞」區塊,用跟成語卡「相似成語/相反成語」一樣的 `.idiom-tag` 圓角標籤樣式(新增了 `.idiom-tag.moreword` 藍色系),點一下標籤就直接唸出那個詞(`speakHighlight`,沒有例句、沒有克漏字,單純聽發音)。**這個欄位是可選的**——`renderCharacterStage()` 裡用 `(c.moreWords && c.moreWords.length)` 判斷要不要渲染這個區塊,沒有這個欄位的字(或欄位是空陣列)就不會顯示這一塊,不會壞掉。

目前 `character1`~`character11` 全部 176 個字都已經加好了 `moreWords`(少數幾個字,像「蝴」「裳」「聰」「匹」「剛」「佩」「柿」「愉」,因為本身能組的常見詞太少、且都已經被 `words[]` 用光,所以沒有額外加,`moreWords` 欄位直接省略)。**加這些詞的時候有做過三項自動驗證,之後如果要再擴充,務必比照辦理**:①每個新詞裡一定要包含這張卡片的主角字(不能不小心塞錯字);②新詞不能跟這個字 `words[]` 裡已經有的詞重複;③同一個字底下新加的詞彼此不能重複。另外**破音字要特別小心讀音**:像「為」「還」「切」「吐」「種」「背」這幾個字有兩種唸法,新增的 `moreWords` 一律只選跟這張卡片 `pinyin` 欄位標示的那個讀音一致的詞,不要混進另一個讀音的詞進去(例如「切」唸 ㄑㄧㄝ 是「切菜」的切,「一切」的切要唸 ㄑㄧㄝˋ,是不同讀音,不能放進 ㄑㄧㄝ 那張卡片的 `moreWords` 裡)。

### 字音字形練習(`charDiscrimination`,2026-08-21 新增)

使用者接著要求「字音字形」練習(國小國語常見的題型:同音字、形似字辨析,例如「己/已/巳」長得很像但讀音不同,「以/已」讀音一樣但字不一樣),而且明確要求**以課文生字為主**,並指定分組方式:第1~3課、第4~6課、第7~9課、第10~12課生字各自集結成一個練習單元。**第12課使用者當時還沒提供內容**(見上面「未完成事項:國字第12課」那節),所以第四個單元目前只涵蓋第10~11課,標題跟 CLAUDE.md 都有清楚註記這件事,等她之後補上第12課生字,要記得回來把第四個單元也一起擴充。

**這個內容不是資料重組,是需要 Claude Code 自己判斷「哪些字真的容易搞混」的內容,錯誤風險比之前的題庫擴充都高**(選錯配對等於教小孩錯的讀音/寫法),所以做法跟之前「隨機重組現有資料」的題庫都不一樣:每一題的目標字(正確答案)都是這176個生字裡的其中一個,但用來當「錯誤選項」的同音字/形似字,是 Claude Code 逐字查證讀音之後手動配對出來的,**不是程式自動配對**。例如：

- 同音字型(字音):「望」「忘」都唸ㄨㄤˋ,「圓」「元」「園」都唸ㄩㄢˊ,「賣」「麥」都唸ㄇㄞˋ(但「買」是ㄇㄞˇ,聲調不同,刻意保留來練習分辨聲調)。
- 形似字型(字形):「以/已/己」長得很像但讀音不同(以=已=ㄧˇ,己=ㄐㄧˇ),「樣/養/羊」都有「羊」的部件但讀音不同。

因為每個單元「真的適合拿來出題」的生字組合有限(不是每個字都找得到自然的同音字/形似字),**沒有硬湊到176個字都出題**,四個單元各挑了15組最有把握、最不會誤導小孩的組合當核心題目,再用跟其他題庫一樣的手法(同一組核心題目、選項順序重新洗牌)展開成 `levelPool` 60題、`poolSampleSize:15`,格式跟其他章節完全一致。

`enabledTabs: ['grammar','game']`(跟數學科同一套模式,沒有故事卡),`category: 'charDiscrimination'`(首頁分類篩選按鈕「🔤 字音字形」,CSS 新增 `.chip-charDiscrimination` 粉紅色系),`ruleType:'mathPattern'` 的規律筆記講解「什麼是字音字形」的概念(不是逐字列出176個字的配對,只講方法)。四個章節:`charDiscrim1`(第1~3課)、`charDiscrim2`(第4~6課)、`charDiscrim3`(第7~9課)、`charDiscrim4`(第10~11課),`unitLabel` 是「字音字形 1~4」。

**驗證方式**:除了跟其他題庫一樣檢查「選項裡有正確答案、選項不重複」之外,額外加了一項這個內容專屬的檢查——**hint 文字裡一定要包含正確答案那個字**(避免手動打字時字打錯或答案設定錯誤導致提示文不對題)。三個聲調/讀音的配對都是 Claude Code 憑既有的注音知識人工核對過的,**如果小孩之後在使用時發現有哪一組配對的讀音其實不對,要立刻回報,這種內容錯了比其他題庫的錯誤更需要優先修正**(其他題庫答錯只是體驗不好,這個如果配錯字音字形會直接教錯)。

### 數學科(九九乘法,2026-08-20 新增)

這是第三個 `subject`,值是 `'math'`(對應首頁上方選單「🔢 數學 Math」)。目前只有一個 `category:'multiplication'`,`enabledTabs: ['story','grammar','game']`(沒有單字卡、拼讀組合、換我說說看)。

這個 category 的三個分頁分別對應到:

- **story 分頁(標籤改成「📊 格子圖」)**:渲染函式是 `renderMultiplicationStage()`,不是共用的 `renderStage()`。用 `activeChapter.multIndex`(0~8,對應 ×1~×9)控制目前顯示第幾個算式,用格子/點點排成 `multiplier` 列 × `n` 欄的陣列圖,讓小孩直接數格子理解乘法的意義。有個「🔄 交換排排看」按鈕會把 `activeChapter.multFlipped` 切換,把陣列轉成 `n` 列 × `multiplier` 欄,格子數量不變,用來直接「看到」交換律(a×b=b×a)。上一頁/下一頁按鈕、整頁朗讀按鈕(readAllBtn)都在 `prevBtn`/`nextBtn`/`readAllBtn` 的 click handler 裡各自加了 `category === 'multiplication'` 的分支去處理,**不是**共用 idiom/character/預設(story)那幾支已有的邏輯。
- **grammar 分頁(標籤改成「🔍 規律心法」)**:共用 `renderRulePanel()`,但多加了一個新的 `ruleType: 'mathPattern'` 分支(跟原本 `soundChart`/`verbMirror` 是平行的第三種)。內容資料放在章節物件的 `patternNotes` 陣列裡,每個元素是 `{heading, body:[...]}`,`body` 裡的字串可以直接寫 `<span class="mult-pattern-eq">...</span>` 這種行內 HTML 來強調算式(這個欄位不會被跳脫處理,因為內容是我們自己寫的,不是使用者輸入)。
- **game 分頁(標籤改成「🎮 過關挑戰」)**:**完全沒有新增引擎邏輯**,直接沿用原本國字生字/英文章節共用的 `renderQuiz()`/`handleAnswer()` 闖關系統(`levels[]` 陣列、`blanks:1`、`before`+`after`+`options`+`correct`+`hint` 的克漏字格式)。混合了「直接背誦題」(`9 × 3 = ___`)、「缺格找答案」(`9 × ___ = 54` 或 `___ × 9 = 45`)、「生活應用題」(文字情境題)、「拆解法練習題」四種類型,全部都是同一套 `blanks:1` 格式,只是 `before`/`after` 的文字內容不同,**不需要另外寫新的題型判斷邏輯**。

**TTS 語音語言的小地雷**:原本判斷要用中文還是英文語音,程式碼裡到處都是 `activeChapter.subject === 'mandarin'` 這種寫法。數學科的內容全部是中文(算式、應用題文字都用國語念),所以額外寫了一個共用函式 `isZhSubject(ch)`(回傳 `ch.subject === 'mandarin' || ch.subject === 'math'`),把原本三處直接寫 `subject === 'mandarin'` 的地方都改成呼叫這個函式。**之後如果又加新的中文科目(不是英文),TTS 語言判斷要記得檢查/改用 `isZhSubject()`,不要漏掉某個地方繼續用 `=== 'mandarin'`。**

**CATEGORY_META 新增了 `multiplication` 分類**(首頁分類篩選按鈕「✖️ 九九乘法」),CSS 新增了 `.chip-multiplication`(綠色系)、`.mult-*` 開頭的一整組樣式(陣列格子圖、規律筆記卡片、交換律提示框)。

目前做了 `mult9`(9 的乘法,深入理解單一乘數用,有格子圖+規律心法+固定 12 關)當第一版試做。使用者之後如果想繼續照這個格式做 `mult6`/`mult7`/`mult8`(她說這幾個是小孩比較容易搞混的),直接複製 `mult9` 的物件格式,改 `id`/`multiplier`/`patternNotes`/`levels` 裡的算式內容就好,`renderMultiplicationStage()`/`renderRulePanel()`/`renderQuiz()` 這些渲染邏輯完全不用動。

### 隨機題庫機制(levelPool,2026-08-20 新增)

使用者接著要求「99乘法表的題目幫我新增50-70題,每次隨機列出題目」,涵蓋範圍是 1~9 全部乘數混在一起(不限於 9),所以新增了第二個數學章節 `multMixed`(「99 乘法綜合練習」),跟 `mult9` 是完全不同的資料結構,重點是**這一個章節沒有固定的 `levels`,而是用 `levelPool` + 每次進入時隨機抽樣**。

- `levelPool`:63 題的大題庫(直接背誦 30 題、缺格找答案 18 題、生活應用題 15 題,`blanks:1` 克漏字格式,跟 `mult9` 的題目格式完全一樣,只是題目更多、乘數不限於 9)。
- `poolSampleSize: 15`:每次要從題庫裡抽幾題出來玩。
- `levels: []`:一開始是空的,**不要手動填內容**,這個欄位是動態產生的。
- 共用函式 `sampleLevelPool(ch)`(在 `CHAPTERS.forEach` 那段附近):用 Fisher-Yates 洗牌把 `ch.levelPool` 打亂,取前 `poolSampleSize` 題,重新編號 `id`(1~15)、重設 `attempts`/`stars`/`done`,寫進 `ch.levels`。這個函式在兩個時機被呼叫:①頁面剛載入時(`CHAPTERS.forEach` 那段,讓首頁卡片一開始就有正確的星星進度可以顯示,不會出現 0/0 的閃爍)、②`enterChapter()` 一開始(每次點進這個章節都重新抽一次,達到「每次進來都不一樣」的效果)。
- `multMixed` 的 `enabledTabs` 只有 `['game']`(沒有 story/grammar),因為題目混合了各種乘數,格子圖跟規律心法只適合針對單一乘數,不適合這種綜合題庫。

**之後如果要擴充題庫、或是想幫其他練習(不限於乘法)也做「每次隨機抽題」的效果,可以直接照這個模式:資料裡放 `levelPool` + `poolSampleSize`、`levels: []`,`sampleLevelPool()` 這個函式完全通用、不用改,只要章節物件有 `levelPool` 欄位就會自動套用。**

**⚠️ 出題資料要注意的地雷**:寫應用題(word problem)的時候,要分清楚哪些數字是「可以隨便變」的(例如一盒糖果有幾顆,不同盒子本來就會不一樣),哪些是「現實世界的固定常數,不能亂帶數字進去」(例如一台腳踏車一定是 2 個輪子、一隻章魚一定是 8 隻腳、一個星期一定是 7 天、一隻手一定是 5 根手指)。生成這批題目時第一版程式沒注意到這點,曾經生出「一台腳踏車有 3 個輪子」這種錯誤事實的題目,後來改成把這幾個「現實常數」寫死、只讓另一個數字(份數)隨機變化才修正。**之後如果再新增這類應用題,先想清楚題目裡的每個數字是不是現實世界的固定事實,不要為了湊隨機變化就讓不該變的數字也跟著變。**

### 第二個數學單元:200 以內的數(numberSense,2026-08-21 新增)

使用者傳了翰林版二上數學(一)第一單元「200 以內的數」的課綱(1-1 數到200、1-2 幾個百幾個十幾個一、1-3 付錢、1-4 數的大小比較),要求四個小節一次全部做。她手邊沒有課本內頁照片,是請 Claude Code 自己上網搜尋(均一教育平台 `junyiacademy.org` 上有明確標示對應翰林版的課程結構)確認每個小節的學習目標範圍後,自己設計例題跟題目做出來的,**不是逐字照抄課本內容**。

跟 `mult9`/`multMixed` 不同,這四個小節**刻意沒有做故事劇場(story)分頁的視覺化**(不是每個小節都做了像 `renderMultiplicationStage()` 那種格子圖):

- `category: 'numberSense'`(CATEGORY_META 新增了這個分類,首頁分類篩選按鈕「🔢 200以內的數」,CSS 新增了 `.chip-numberSense`,橘黃色系)。
- 四個章節的 `enabledTabs` 都只有 `['grammar', 'game']`,**完全沒有新增任何渲染邏輯**——grammar 分頁沿用 `ruleType: 'mathPattern'`(跟 `mult9` 共用同一支 `renderRulePanel()` 邏輯,只是 `patternNotes` 換成位值/錢幣/比大小的內容,用 emoji 模擬積木/鈔票當視覺輔助,借用了 `mult-pattern-eq`/`mult-pattern-box` 這兩個原本給乘法用的 CSS class,因為样式本身跟乘法無關,拿來給別的數學概念用完全沒問題);game 分頁沿用 `renderQuiz()`/`handleAnswer()`,每章固定 10 關,`blanks:1` 克漏字格式,跟 `mult9` 完全一樣的資料結構。
- 四個章節:`numTo200`(數到200)、`placeValue200`(幾個百幾個十幾個一)、`payingMoney`(付錢)、`numberCompare200`(數的大小比較),`unitLabel` 依序是「數學 3」~「數學 6」(接續 `mult9`=數學1、`multMixed`=數學2)。

**之後如果想幫這四個小節之一補做視覺化的 story 分頁(例如幫「幾個百幾個十幾個一」做真的積木圖,而不是 emoji 文字描述),可以參考 `renderMultiplicationStage()` 的寫法另外寫一支新函式,再把該章節的 `enabledTabs` 加回 `'story'`——但目前使用者沒有明確要求視覺化,是 Claude Code 為了控制範圍/風險自己決定先用 emoji + 文字筆記的方式做,不是課綱本身要求的形式,如果她覺得不夠直觀想要真的畫圖,可以再加。**

### 第三~五個數學單元:加減法/公分/容量(2026-08-21 新增)

同一天使用者又連續傳了翰林版二上數學(二)(二、二位數的加減法:2-1加法、2-2減法、2-3等於大於小於)、(三)(三、認識公分:3-1個別單位、3-2認識公分、3-3量一量畫一畫、3-4長度的加減)、(四)(四、加減應用,**但只給了 4-1 加法和減法的關係,沒有給 4-2 解題和驗算**)、(五)(五、容量:5-1認識容量、5-2容量的比較)。做法跟「200 以內的數」那次完全一樣的模式:自己上網查均一教育平台確認翰林版對應的學習目標範圍,自己設計例題出題,不是照抄課本;每個小節一個章節,`enabledTabs: ['grammar','game']`,沒有另外寫渲染邏輯,完全沿用 `renderRulePanel()`(`ruleType:'mathPattern'`)+ `renderQuiz()`。

新增了兩個 CATEGORY_META 分類:

- `addSubtract`(➕➖ 加減法,`.chip-addSubtract` 藍綠色系):`addTwoDigit`(2-1二位數的加法,數學7)、`subtractTwoDigit`(2-2二位數的減法,數學8)、`compareSums`(2-3等於大於和小於——**不是單純比較兩個數字**,是比較兩個算式的計算結果,跟 `numberSense` 分類裡的 `numberCompare200` 是不同章節、不同重點,注意不要搞混,數學9)、`addSubtractRelation`(4-1加法和減法的關係/加減互逆+括號算式,數學14)。
- `length`(📏 認識公分,`.chip-length` 黃色系):`unitLen1`(3-1個別單位,數學10)、`cm1`(3-2認識公分,數學11)、`measureDraw`(3-3量一量畫一畫——包含「斷尺」讀法,即物品沒有對齊0刻度時要用「右邊刻度−左邊刻度」算長度,數學12)、`lengthAddSub`(3-4長度的加減,包含量超過尺長的東西要分段量再相加,數學13)。這四個小節因為牽涉到直尺、量測、畫線段等實際操作,**目前也是完全用文字/emoji 描述題目情境,沒有做真的可以拖拉量測的直尺互動元件**,跟「200以內的數」那次一樣是 Claude Code 為了控制風險自己決定的簡化版本,如果使用者覺得不夠直觀,可以之後再考慮做真的直尺互動。
- `capacity`(🧴 容量,`.chip-capacity` 靛藍色系):`capacity1`(5-1認識容量,數學15)、`capacity2`(5-2容量的比較,數學16)。這兩章完全是概念性題目(容量是什麼、外表不能決定容量、個別單位/間接比較法、容量守恆),沒有算式計算,`options` 大多是完整句子而不是數字,跟 `payingMoney` 裡「夠/不夠」那種問答式選項是同一種寫法。

**⚠️ 未完成事項:數學(四)加減應用只做了 4-1**,使用者只提供了「4-1 加法和減法的關係」,均一教育平台顯示這個單元其實還有「4-2 解題和驗算」(加減互逆用在驗算、含括號算式解題),**目前 4-2 還沒有做**,如果她之後傳 4-2 的課綱或明確要求,再照同樣模式加一個新章節(建議 id 取 `checkAddSubtract` 或類似名稱,`unitLabel` 接續用「數學17」,`category` 沿用 `addSubtract`)。

### 全站題目隨機化改造(2026-08-21 新增,重要)

使用者要求「所有題目每次進去都是隨機不同的,題庫要有幾十題再輪流測驗」,範圍是**整個網站、所有科目的所有闖關遊戲**(英文 6 章、國語 12 章、數學 15 個新章節,`mult9` 也算在內,`multMixed` 本來就已經是題庫模式了)。做法完全套用 `multMixed` 已經建立好的機制(見上面「隨機題庫機制」那節):每個章節原本的固定 `levels` 陣列,都改成 **`levelPool`(60 題題庫)+ `poolSampleSize: 15`(每次抽15題)+ `levels: []`**,`sampleLevelPool()` 這支函式完全沒有改,是通用的,只要章節物件有 `levelPool` 欄位就會自動套用,所以這次改造**完全是資料層面的變動,沒有動任何渲染邏輯**。

執行方式是寫了幾支一次性的 Node.js 產生器腳本(跑完就刪掉了,不在專案裡),分三批做:

- **數學 15 章**:程式化產生(算式類直接用程式算出正確答案,保證不會算錯;像「200以內的數」「認識公分」「容量」這種概念類的,用同一套情境模板帶入不同數字/物件產生變化)。
- **國語 12 章(成語+生字1~11課)**:**直接從章節資料裡原本就有的 `characters`/`idioms` 陣列重組出新題目**,例如生字的「部首」「筆畫」本來就有欄位,「造詞」直接借用 `words[].sentence` 例句挖空,破音字直接借用 `polyphonic[].example` 例句——**完全沒有自己編新的中文內容**,只是把已經審過、正確的資料排列組合成更多題目,風險最低。
- **英文 6 章(Phonics 1~5 + 農場故事 unit11)**:Phonics 是借用 `soundGroups[].words` 現有單字重新排列組合(哪個字用哪個發音組合,原本資料就有)。unit11 比較特別,原本只有 4 關手工設計的文法題(有/have/has、想要/want/wants、需要/need/needs 三組動詞 + doesn't 魔法鏡),**這 4 關原封不動保留在新題庫裡**,另外用同一套文法規則(單數第三人稱動詞要加s、doesn't後面動詞要變回原形)自動生成了不同主詞(She/He/The farmer/Mrs. Flores/I/We/You/They)+不同受詞(cow/milk/comic book/board game/stickers/doll/fancy hat/money/food,這些字都是原本 `vocab`/`homework` 欄位裡就有的,沒有新增字彙)組合出的句子,湊到 60 題。

**⚠️ 品質控管:每一批都寫了「獨立驗證」的檢查腳本,不是只信任產生題目的同一段程式碼**,包括:選項裡一定要有正確答案、選項不能重複、不能出現負數當選項(2年級小孩沒學過負數)、數學算式的正確答案是重新用程式算一次來對答案(不是憑印象抄數字)、英文文法題是重新從句子的主詞去反推該用哪個動詞形式來檢查(不是只檢查選項格式對不對)、Phonics題目是重新對照 `soundGroups` 資料確認拼出來的單字真的在那個發音分組裡。**跑驗證的過程中真的抓到並修正了兩個數學算式打錯的 bug**(一個是 `42+15` 誤寫成 `=27`,另一個是 `2張500元+1張100元` 誤寫成 `1200` 應該是 `1100`),所以這個「獨立驗證」步驟不是多餘的形式,之後如果還要再擴充任何題庫,務必比照這個做法,不要只憑肉眼檢查就上線。

**⚠️ 另一個上線後才發現的地雷:國字生字的「部首」題目,選項字串技術上不重複,但視覺上會混淆。** 使用者截圖回報過一題:「吃」的部首選項出現「口部」跟「囗部」兩個看起來幾乎一樣的方框字(口 U+53E3 是小口,囗 U+56D7 是大口/圍字部,兩個是不同字但小孩子(甚至大人)在按鈕上很難分辨)。程式當初的重複檢查只檢查字串是否完全相同,沒檢查「視覺上容不容易搞混」,所以沒抓到。已經手動把 `character` 系列生字資料裡兩題受影響的選項換掉(`吐`、`吃` 這兩題原本的 `囗部` 選項換成別的部首)。**已知還有其他容易混淆的部首組合,之後如果要重新產生或新增生字部首題目,decoy(錯誤選項)要避開讓下面這幾組同時出現在同一題裡:**`口部`/`囗部`、`日部`/`曰部`、`人部`/`入部`/`人部(亻)`。

改完之後用 Playwright 對全站 34 個闖關章節(33個新改的+`multMixed`)逐一進入測試過:每章 `levelPool` 都是60題(`multMixed`維持它原本的63題)、每次進入都抽15題、連續兩次進入題目確實不一樣(證明隨機化真的生效)、答對正確答案真的會出現星星回饋,全部通過、瀏覽器主控台沒有任何錯誤。

**之後如果要再幫某個章節擴充題庫或調整抽題數量,直接改該章節的 `levelPool` 陣列內容或 `poolSampleSize` 數字就好,不用動任何函式邏輯。** 如果之後又新增全新章節(不是擴充既有的),記得從一開始就直接用 `levelPool` 格式,不要先寫固定 `levels` 之後又要重做一次。

## 進度自動存檔 + 今日精選挑戰 + 錯題本 + 學習地圖擴大(2026-08-21 新增)

使用者請 Claude Code 全站檢查一遍,檢查完之後請她推薦幾個「讓練習更有趣」的方向,她選了「今日精選混合小測驗」「錯題本」「進度地圖視覺化(擴大現有功能)」這三個。開始做之前發現一件重要的事:**這個網站原本完全沒有把星星進度存起來**,所有 `stars`/`attempts`/`done` 都只活在瀏覽器分頁的記憶體裡,重新整理頁面就全部歸零。這三個新功能如果不先解決這件事,做出來意義不大(錯題重新整理就不見、進度地圖每次打開都是0),所以**先加了一層 localStorage 存檔機制當地基**,再蓋上面三個功能。

### 存檔機制(`PROGRESS` / localStorage)

- `PROGRESS_KEY = 'kidsDailyPractice_progress_v1'`,存在瀏覽器的 `localStorage` 裡(不用登入、不用網路,存在她自己的瀏覽器裡;`file://` 本機開啟跟 GitHub Pages 部署都測試過可以正常存取)。
- 結構:`PROGRESS = { stars: {...}, mistakes: {...}, dailyChallengeDates: [...] }`。
  - `stars[章節id][questionKey] = 這題拿到的星星數(1~3)`
  - `mistakes[章節id][questionKey] = { wrongCount, lastWrongAt }`(存在代表「還沒複習對」,答對後會自動從這裡刪掉)
  - `dailyChallengeDates = ['2026-08-21', ...]`(記錄哪幾天玩過「今日精選挑戰」,只是給首頁小提示用,不會鎖住不能重玩)
- **`questionKey(l)` 是關鍵**:因為 `levelPool` 章節每次進入都會用 `sampleLevelPool()` 重新隨機抽15題,題目的 `id` 每次都會被重新編號(1~15),**不能拿 `id` 當作題目的固定身分**,所以改用「內容指紋」當 key:`before+after+correct`(兩格填空是 `before+mid+after+correct`)。這樣同一題不管被抽到第幾關、抽了幾次,存檔查出來都是同一筆記錄。
- `restoreLevelStars(ch)`:每次 `sampleLevelPool()` 抽完題之後(章節初始化時、`enterChapter()` 時)都會呼叫這個函式,把這次抽到的15題裡,凡是 `PROGRESS.stars` 裡有記錄的都把 `l.stars`/`l.done` 補回去,所以重新整理、重新進入章節,已經拿過的星星都還在。
- `handleAnswer()` 裡的存檔邏輯用的是 `l._originChapterId || activeChapter.id`,不是直接寫死 `activeChapter.id`——這是為了讓「今日精選挑戰」「錯題複習」這種**混合來源**的關卡,答對/答錯時能存回**題目真正出處的章節**,而不是存到一個假的合成章節 id 底下。

### 今日精選混合小測驗(`startDailyChallenge()`)

點首頁「🎯 今日精選挑戰」按鈕,會從**全站所有章節的 `levelPool`**(不分英文/國語/數學)隨機抽 10 題混在一起(`DAILY_CHALLENGE_COUNT = 10`),組成一個**合成章節物件**(不是真的加進 `CHAPTERS` 陣列,只是一個形狀很像的暫時物件),直接沿用既有的 `renderQuiz()`/`handleAnswer()`/`renderLevelTabs()` 闖關引擎,只是分頁只留 `game`,`story`/`vocab`/`grammar`/`blend`/`homework` 全部隱藏(這幾個分頁的東西對混合題目沒有意義)。

因為題目混合中英文,原本用 `isZhSubject(activeChapter)` 判斷語音語言的地方行不通(合成章節沒有單一 `subject`),所以每一題會多帶一個 `_lang`(`'zh-TW'` 或 `'en-US'`,抽題當下就從題目原本所屬章節算好),新增了 `quizLang(ch, l)` 這個函式,**優先看 `l._lang`,沒有才退回原本的 `isZhSubject(ch)`**,`renderQuiz()` 的「聽聽看答案」按鈕跟 `handleAnswer()` 裡兩處語音判斷都已經改用這個函式。

完成全部10題(進到 `renderQuiz()` 的「Mission Complete」畫面)時,如果 `activeChapter.category === 'dailyChallenge'`,會呼叫 `markDailyChallengeDoneToday()` 把今天日期記進 `PROGRESS.dailyChallengeDates`,首頁按鈕旁邊會出現「✅ 今天玩過了」的小提示文字(`updateDailyChallengeNote()`)——**這只是溫和提示,不會鎖住不能重玩**,使用者當初的方向是「不設壓力式連續打卡」。

### 錯題本(`startMistakeReview()`)

首頁「📕 錯題本」按鈕,右上角會有一個紅色數字徽章顯示目前全站累積多少題還沒複習對(`updateMistakeBookBadge()`,每次答對/答錯後都會重新算)。點下去會呼叫 `collectMistakeQuestions()`:把 `PROGRESS.mistakes` 裡每個章節、每個 `questionKey`,拿去**該章節目前的 `levelPool` 裡重新查一次原始題目內容**(不是存一份題目快照)——這樣做的好處是,如果之後修正了某題的文字/答案錯誤,錯題本裡看到的會自動是修正後的版本,不會卡住舊的錯誤內容。查不到的(代表題目後來被刪掉或改到匹配不上)就順便把這筆過期紀錄清掉。收集完一樣組成合成章節、丟進 `renderQuiz()` 引擎;**答對某一題就會自動把它從 `PROGRESS.mistakes` 刪掉**(`handleAnswer` 正確分支本來就有這段邏輯),所以錯題本是「越練越少」、會自己清空,不需要另外寫「標記已複習」的按鈕。如果目前沒有任何錯題,點下去只會跳出一個溫馨的 `alert`,不會進入空白畫面。

### 學習地圖擴大(`renderProgressOverview()`)

原本右下角 📊 按鈕開的「Phonics 學習地圖」是 2026-08-19 手工寫死的靜態內容,只涵蓋 Phonics 發音範圍。這次**沒有砍掉重寫**,是在同一個 modal 最上面新增了一個動態區塊 `#rmOverview`(標題也從「🔤 Phonics 學習地圖」改成「📊 學習地圖」,原本的 Phonics 詳細內容往下移,中間加一條分隔線),原本手寫的 Phonics 內容原封不動保留在下面。

新的 `#rmOverview` 是**完全動態算出來的**,不是手寫資料:對英文/國語/數學三科,把該科每個章節的 `chapterProgress(ch)` 算出來——**注意這裡的「總分」用的是章節的完整 `levelPool.length * 3`(全部60題左右可能拿到的星星上限),不是首頁卡片顯示的「這次抽到的15題」那個小分母**,所以首頁卡片的星星數字(每次進入會因為抽到不同15題而波動)跟這個學習地圖裡的星星數字(全題庫累積,只會往上加不會因為換題目而變動)**本來就會不一樣,這是刻意的設計,不是 bug**——首頁卡片是「這次要不要玩」的即時小分數,學習地圖是「整體學會多少」的長期總覽。每次點開 📊 按鈕都會重新呼叫 `renderProgressOverview()`,保證看到的一定是最新資料。

**這四個東西(存檔機制、今日挑戰、錯題本、學習地圖擴大)都用 Playwright 測試過**,包括真的呼叫 `page.reload()` 確認星星在重新整理後真的還在、確認 `file://` 本機開啟時 localStorage 也正常運作(不只 GitHub Pages 部署版本)、確認錯題答對後真的會從錯題本消失、確認今日挑戰完成後首頁真的會顯示提示文字、確認學習地圖數字沒有 NaN/undefined。全站 34 個章節的既有闖關功能也重新回歸測試過一次,確認這次改動沒有連帶弄壞任何東西。

## 內容現況(截至最後更新)

### 英文(subject: 'english')
- `unit11`:一般文章,`category:'story'`,6 個分頁全開
- `phonics1` ~ `phonics5`:Phonics 章節,`category:'phonics'`,6 個分頁全開(其中會用到 `blend` 分頁跟 `blends` 資料)

### 國語(subject: 'mandarin')
- `idiom1`:成語,`category:'idiom'`,`enabledTabs:['story','game']`
- `character1` ~ `character11`:康軒二上國字生字第 1~11 課,`category:'character'`,`enabledTabs:['story','game']`
- `charDiscrim1` ~ `charDiscrim4`:字音字形練習,依生字課次分組(第1~3課/4~6課/7~9課/10~11課),`category:'charDiscrimination'`,`enabledTabs:['grammar','game']`。第12課還沒有生字資料,第四個單元暫時只有10~11課,細節見上面「字音字形練習」那節。

### 數學(subject: 'math')
- `mult9`:9 的乘法,`category:'multiplication'`,`enabledTabs:['story','grammar','game']`,深入理解單一乘數用(格子圖+規律心法)。`mult6`/`mult7`/`mult8` 還沒做,等使用者想繼續做的時候再照同樣格式加。
- `multMixed`:99 乘法綜合練習,`category:'multiplication'`,`enabledTabs:['game']`,涵蓋 1~9 全部乘數。
- `numTo200`/`placeValue200`/`payingMoney`/`numberCompare200`:翰林版二上數學第一單元「200 以內的數」四個小節,`category:'numberSense'`,`enabledTabs:['grammar','game']`。細節見上面「第二個數學單元:200 以內的數」那節。
- `addTwoDigit`/`subtractTwoDigit`/`compareSums`/`addSubtractRelation`:翰林版二上數學第二單元「二位數的加減法」(2-1~2-3)+第四單元「加減應用」的 4-1,`category:'addSubtract'`,`enabledTabs:['grammar','game']`。4-2 還沒做,見下面「未完成事項」。
- `unitLen1`/`cm1`/`measureDraw`/`lengthAddSub`:翰林版二上數學第三單元「認識公分」四個小節(3-1~3-4),`category:'length'`,`enabledTabs:['grammar','game']`。
- `capacity1`/`capacity2`:翰林版二上數學第五單元「容量」兩個小節(5-1~5-2),`category:'capacity'`,`enabledTabs:['grammar','game']`,純概念題沒有計算。
- 細節見上面「第三~五個數學單元」那節。

**⚠️ 上面這些章節現在全部都是 `levelPool`(60題左右)+ `poolSampleSize:15` 的隨機抽題模式,不再是固定關卡數了**,詳見上面「全站題目隨機化改造」那節。之後看章節資料時,不要再假設是「固定幾關」,先看有沒有 `levelPool` 欄位。

### ⚠️ 未完成事項:國字第 12 課

使用者之前提供的「第 12 課」內容,跟已經做好的 `character6`(第 6 課)完全重複,已經跟她反應過這件事,**目前第 12 課還沒有做**,在拿到正確的第 12 課內容之前不要自己編內容硬做。如果她之後傳新的第 12 課資料,依照 `character1`~`character11` 一樣的物件格式加進去就可以了(15 字/15 關的慣例可以參考,但不是硬性規定,配合實際課文生字數量即可)。

## 檔案大小:要不要拆成多個檔案?

使用者問過這個問題(加了數學科之後擔心「都塞在同一個 index.html,以後資料越來越多怎麼辦」)。目前的結論是:**先不要拆**,原因記錄在這裡,之後如果又被問到同樣的問題,不用重新想一次。

- 2026-08-20(剛加完數學科 `mult9`)當下的檔案大小基準:約 280KB / 5473 行。
- 2026-08-21(全站題目隨機化改造,把 33 個章節的固定題目都擴充成 60 題題庫)之後:約 760KB / 7882 行。漲了快 3 倍,主要是因為每章題庫從 10~17 題左右擴充到 60 題。
- 2026-08-21(加完進度存檔+今日挑戰+錯題本+學習地圖+4個字音字形單元)之後:約 **860KB / 8707 行**。**目前還在安全範圍內(門檻是 1.5MB / 20000 行),但已經超過基準的一半了,之後如果還要繼續擴充題庫或加新章節,記得留意這個數字有沒有接近門檻。**
- 不拆成多檔案的原因:英文/國語/數學三科目前共用同一套「引擎」(語音朗讀 `speakHighlight`、分頁切換、闖關遊戲 `renderQuiz`/`handleAnswer`、CSS 樣式)。因為這個專案刻意不用 build 工具(沒有 npm/webpack),如果拆成多個 HTML 檔案,這些共用邏輯必須整份複製貼上到每個檔案裡,之後每次修 bug 都要記得改好幾份、很容易漏改讓檔案內容兜不起來。純資料成長(章節內容變多)對瀏覽器載入速度幾乎沒有影響,即使長到 1MB 等級都還是瞬間載入的等級,不是網頁效能會遇到的瓶頸。

**⚠️ 提醒機制:如果之後修改這個檔案時,發現 `index.html` 的大小已經成長到明顯比這個基準大很多(粗抓一個門檻:超過 1.5MB,或行數超過 20000 行),請主動跟使用者提一下,不用等她自己發現或詢問。** 到時候可以考慮的折衷做法是:把「引擎」(共用的 JS/CSS)留在 `index.html` 裡不動,只把「資料」(`CHAPTERS` 陣列裡各科章節的內容)拆成幾個獨立的 `.js` 檔案,用 `<script src="...">` 載入 —— 這樣共用邏輯還是只有一份,不會有多份同步的風險,但代價是她每次更新時,上傳的檔案會從「一個檔案」變成「一個檔案+幾個資料檔案」,部署流程會稍微變複雜一點,所以只有在真的有必要時才做這個改動,不要為了「感覺比較整齊」就提前拆。

## 測試慣例(如果要改程式碼,建議照這個流程驗證)

1. 用正規表達式把 `<script>...</script>` 裡的內容抽出來,跑 `node --check` 確認語法沒錯
2. 用 Playwright(headless Chromium)開頁面測試,記得先 mock 掉 `speechSynthesis.getVoices`/`speak`/`cancel`,不然語音相關的程式碼在無頭瀏覽器裡會出錯。**⚠️ mock 的時候不要整個 `window.speechSynthesis = {...}` 重新賦值,這個屬性在測試環境裡是不可重新賦值的,賦值會被靜默忽略,App 還是會呼叫到瀏覽器原生的 SpeechSynthesis 導致奇怪的錯誤。正確做法是直接改寫原生物件身上個別的方法**,例如 `speechSynthesis.speak = (u) => {...}`(不要 `window.speechSynthesis = {speak:...}`),細節見上面「TTS『唸出這一頁』導覽中斷機制」那節。
3. **點擊測試務必用 `element.dispatchEvent(new MouseEvent('click', {bubbles:true, cancelable:true}))`,不要用 Playwright 原生 `.click()` 或 JS 的 `btn.click()`。** 這兩種在某些多步驟測試腳本裡曾經不可靠地沒有觸發 event listener(原因不明,懷疑是這個環境本身的怪癖,不是網頁本身的 bug),用 `dispatchEvent` 目前為止 100% 可靠。
4. 測完記得清掉暫存的測試檔案

## 跟使用者溝通時的注意事項

- 她不是工程師,說明改動時用白話文,避免只丟術語
- 她習慣傳截圖描述問題,回覆時直接針對截圖內容處理
- 回覆時避免過度使用條列式/標題格式(除非在寫這種技術文件),平常對話盡量用自然的段落
- 每次有實質內容更新,別忘了:①把 `index.html` 寫進她本機的專案資料夾 ②版本號 `#appVersion` 往上加一碼 ③提醒她 commit + push ④push 完提醒她強制重新整理瀏覽器再看
