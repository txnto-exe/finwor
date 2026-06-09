ИНСТРУКЦИЯ ДЛЯ ИИ — ПРОЕКТ FINWOR
====================================

ФАЙЛ ПРОЕКТА
Один файл: C:\Users\mosoy\Desktop\FinWor\game.html
Весь код (HTML + CSS + JS) в одном месте. Никаких зависимостей.

GitHub Pages (живая версия): https://txnto-exe.github.io/finwor/game.html
Репозиторий: https://github.com/txnto-exe/finwor
Git настроен, токен в remote URL. Пуш: git -C "C:/Users/mosoy/Desktop/FinWor" add game.html && git -C "C:/Users/mosoy/Desktop/FinWor" commit -m "..." && git -C "C:/Users/mosoy/Desktop/FinWor" push

ВАЖНО: ПЕРЕД ПУШЕМ — ВСЕГДА СПРАШИВАТЬ ОДОБРЕНИЕ У ПОЛЬЗОВАТЕЛЯ.

ТЕКУЩЕЕ СОСТОЯНИЕ
- Уровней: 60 (все валидны, проверены)
- Следующие уровни: 61–70 (pipeline.js готов)
- Для генерации: node pipeline.js --phase поляна --count 10

ЭТАПЫ ИГРЫ (PHASES в коде)
🦠 Бактерия  — уровни 1–10
🐛 Червь     — уровни 11–25
🐟 Рыба      — уровни 26–50
🦎 Ящерица   — уровни 51–100
🦕 Динозавр  — уровни 101–175
🐒 Обезьяна  — уровни 176–250
🧍 Человек   — уровни 251–350
🧬 Мутант    — уровни 351–450
🦾 Киборг    — уровни 451–575
🤖 Робот     — уровни 576–1000

РАЗМЕРЫ СЕТОК ПО ЭТАПАМ
Бактерия/Червь  — слова 3–4 буквы,  сетки 3×3..4×4 (9–16 кл.)
Рыба/Ящерица    — слова 5–6 букв,   сетки 5×5..5×6 (25–30 кл.)
Динозавр/Обезьяна — слова 7 букв,   сетки 5×7 (35 кл.), шаблон TY3
Человек/Мутант  — слова 8 букв,     сетки 5×8 (40 кл.), шаблон TZ3
Киборг/Робот    — слова 9–10 букв,  сетки 6×9..6×10 (54–60 кл.)

МЕХАНИКА ИГРЫ
- Сетка N×M букв, игрок свайпает по соседним клеткам (←→↑↓, без диагоналей)
- Найти все слова уровня → уровень пройден (+5 монет)
- Любое другое слово из словаря VOCAB → архив (+3 монеты)
- Каждые 20 архивных слов → +30 монет бонус
- Подсказка: 50 монет | Найти слово: 120 монет | Пропуск: 250 монет

СТРУКТУРА УРОВНЕЙ В КОДЕ
Массив LEVELS, каждый уровень — объект:
{ rows: N, cols: M, grid: ["А","Б",...], words: ["СЛОВО1","СЛОВО2",...] }

grid — массив букв длиной rows×cols
Индекс i: строка = floor(i/cols), столбец = i%cols
Смежность adj4(a,b): |row(a)-row(b)| + |col(a)-col(b)| == 1

ГЛАВНОЕ ПРАВИЛО МАРШРУТОВ
- Нельзя повторять одну форму больше 2 раз в одном уровне
- Последний ряд НЕ является единым словом — разделён между минимум двумя словами
- Пути не пересекаются, все клетки покрыты (100% coverage)

ПРОВЕРЕННЫЕ ШАБЛОНЫ

Шаблон TA (5×5, 25 кл., 5 слов × 5 букв):
  W1: [0,1,2,3,4]        →→→→
  W2: [9,8,7,6,5]        ←←←←
  W3: [10,15,20,21,16]   ↓↓→↑
  W4: [11,12,13,14,19]   →→→↓
  W5: [24,23,22,17,18]   ←←↑→

Шаблон TC (5×6, 30 кл., 5 слов × 6 букв):
  W1: [0,6,12,18,24,25]      ↓↓↓↓→
  W2: [5,4,3,2,1,7]          ←←←←↓
  W3: [8,9,10,11,17,16]      →→→↓←
  W4: [13,14,15,21,20,19]    →→↓←←
  W5: [26,27,28,29,23,22]    →→→↑←

Шаблон TCC (5×6, 30 кл., 5 слов × 6 букв):
  W1: [0,6,12,13,14,15]      ↓↓→→→
  W2: [5,4,3,2,1,7]          ←←←←↓
  W3: [8,9,10,11,17,16]      →→→↓←
  W4: [22,23,29,28,27,21]    →↓←←↑
  W5: [18,19,20,26,25,24]    →→↓←←

Шаблон TY3 (5×7, 35 кл., 5 слов × 7 букв):
  W1: [0,7,14,21,28,29,22]        ↓↓↓↓→↑
  W2: [6,5,4,3,2,1,8]             ←←←←←↓
  W3: [9,10,11,12,13,20,27]       →→→→↓↓
  W4: [15,16,17,18,19,26,25]      →→→→↓←
  W5: [34,33,32,31,30,23,24]      ←←←←↑→

Шаблон TZ3 (5×8, 40 кл., 5 слов × 8 букв):
  W1: [0,8,16,24,32,33,25,26]     ↓↓↓↓→↑→
  W2: [7,6,5,4,3,2,1,9]           ←←←←←←↓
  W3: [10,11,12,13,14,15,23,22]   →→→→→↓←
  W4: [17,18,19,20,21,29,30,31]   →→→→↓→→
  W5: [34,35,27,28,36,37,38,39]   →→↑→↓→→

АВТОМАТИЧЕСКАЯ ГЕНЕРАЦИЯ GRID
  function buildGrid(words, paths, size) {
    const g = new Array(size).fill('?');
    for (let w = 0; w < words.length; w++)
      for (let k = 0; k < words[w].length; k++) g[paths[w][k]] = words[w][k];
    return g;
  }

ПРАВИЛА ПОСТРОЕНИЯ УРОВНЯ
1. Выбрать размер сетки по этапу (см. выше)
2. Каждая клетка ровно в одном слове (пути не пересекаются, все клетки покрыты)
3. Маршруты разнообразные (не одна форма подряд, макс 2 одинаковых)
4. Последний ряд разделён между минимум двумя словами
5. Проверить grid[path[k]] == word[k] для каждого слова

КАК ДОБАВЛЯТЬ В КОД
Добавить объект в конец массива LEVELS перед ];
  { rows:5, cols:7,
    grid:["Ж","А","В",...],
    words:["СЛОВО1","СЛОВО2",...] },

ПОРЦИЯ ДОБАВЛЕНИЯ — СТРОГО 10 УРОВНЕЙ ЗА РАЗ
После каждых 10 уровней написать "готово" и остановиться.
Ждать команду "продолжай" — только тогда делать следующие 10.

PIPELINE (автогенерация уровней)
Файл: C:\Users\mosoy\Desktop\FinWor\pipeline.js
Использование: node pipeline.js --phase <этап> --count <кол-во>
Словарь: C:\Users\mosoy\Desktop\FinWor\wordbank.json

ДЕПЛОЙ
После изменений (только после одобрения пользователя!):
git -C "C:/Users/mosoy/Desktop/FinWor" add game.html
git -C "C:/Users/mosoy/Desktop/FinWor" commit -m "описание"
git -C "C:/Users/mosoy/Desktop/FinWor" push
Изменения появятся на GitHub Pages через ~1 минуту.

ВАЛИДАЦИЯ (запускать после любых изменений уровней)
node -e "
const fs=require('fs');
const html=fs.readFileSync('C:/Users/mosoy/Desktop/FinWor/game.html','utf8');
const LEVELS=eval(html.match(/const LEVELS\s*=\s*(\[[\s\S]*?\n\s*\]);/)[1]);
function row(i,c){return Math.floor(i/c);}
function col(i,c){return i%c;}
function adj(a,b,c){return Math.abs(row(a,c)-row(b,c))+Math.abs(col(a,c)-col(b,c))===1;}
function sh(p,c){return p.slice(1).map((v,i)=>{const dr=row(v,c)-row(p[i],c),dc=col(v,c)-col(p[i],c);return dr===0?(dc>0?'R':'L'):(dr>0?'D':'U');}).join('');}
function findPath(grid,cols,word,used){
  const n=grid.length;
  function dfs(idx,path,vis){
    if(idx===word.length)return path.slice();
    const last=path[path.length-1];
    for(let next=0;next<n;next++){
      if(vis.has(next)||used.has(next))continue;
      if(grid[next]!==word[idx])continue;
      if(path.length>0&&!adj(last,next,cols))continue;
      vis.add(next);path.push(next);
      const r=dfs(idx+1,path,vis);
      if(r)return r;
      path.pop();vis.delete(next);
    }
    return null;
  }
  for(let s=0;s<n;s++){
    if(grid[s]!==word[0]||used.has(s))continue;
    const r=dfs(1,[s],new Set([s]));
    if(r)return r;
  }
  return null;
}
const issues=[];
LEVELS.forEach((lv,idx)=>{
  const{rows,cols,grid,words}=lv;
  const used=new Set();const shapes=[];
  for(const w of words){
    const p=findPath(grid,cols,w,used);
    if(!p){issues.push('Lv'+(idx+1)+' НЕТ ПУТИ: '+w);continue;}
    p.forEach(i=>used.add(i));shapes.push(sh(p,cols));
  }
  if(used.size!==rows*cols)issues.push('Lv'+(idx+1)+' покрытие: '+used.size+'/'+rows*cols);
  const sc={};shapes.forEach(s=>{sc[s]=(sc[s]||0)+1;});
  for(const[s,cnt]of Object.entries(sc))if(cnt>2)issues.push('Lv'+(idx+1)+' форма x'+cnt+': '+s);
});
console.log('Уровней:',LEVELS.length);
if(issues.length===0)console.log('Все OK!');
else issues.forEach(x=>console.log(x));
"

СЛОВАРЬ
const VOCAB — Set из ~1730 русских слов.
Используется только для архивных слов (не для целевых слов уровней).
Целевые слова уровня не обязаны быть в VOCAB.

ТЕМЫ
7 тем: dark, light, ocean, forest, violet, crimson, gold
dark/light — бесплатно
ocean=уровень 50, forest=80, violet=120, crimson=160, gold=200 (или 500 монет)
Объект темы: { id, unlock, bg, cell, sel, stx, used, txt, accent, confetti[] }

БОНУСНЫЕ УРОВНИ
- Запускаются после уровней 9, 19, 29, 39...
- 60 секунд, любое слово из VOCAB = +5 монет
- Данные: массив BONUS_LEVELS[{phase:0/1/2, rows, cols, words:[], grid:[]}]

РЕЖИМ РАЗРАБОТЧИКА (ADMIN)
- Активация: нажать логотип на экране меню 5 раз (2 сек таймаут)
- Функции: ∞ монеты, разблокировать все уровни, прыжок к уровню, сброс прогресса

ВАЖНЫЕ JS ФУНКЦИИ
loadLevel()      — загружает уровень
accept(w, cells) — засчитывает найденное слово
complete()       — уровень пройден (+5 монет, конфетти)
archiveWord(w)   — архивное слово (+3 монеты)
findPath(word)   — ищет путь для слова в текущей сетке
submit()         — проверяет ввод игрока
tHint()          — подсказка (пульсация первой клетки)
applyTheme(id)   — применяет тему, обновляет CSS-переменные
