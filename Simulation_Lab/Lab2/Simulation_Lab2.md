\## Комп'ютерні системи імітаційного моделювання

\## СПм-24-3, \*\*Амосов Дмитро Олександрович\*\*

\### Лабораторна робота №\*\*2\*\*. Редагування імітаційних моделей у середовищі NetLogo



<br>



\### Варіант 1, модель у середовищі NetLogo:

\[Traffic Grid](http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Social%20Science/Traffic%20Grid.nlogo)



<br>



\### Внесені зміни у вихідну логіку моделі, за варіантом:



\*\*Додано жовтий сигнал світлофора.\*\*

Світлофори тепер перемикаються не миттєво з зеленого на червоний, а мають проміжну фазу "жовтий". Для цього використовується параметр `yellow-duration`.

Якщо до перемикання світлофора залишається менше тактів, ніж відсоток від `ticks-per-cycle` який вказано у параметрі `yellow-duration`, вмикається жовтий колір.

Розраховується, скільки тактів `ticks-until-switch` залишилося до планового перемикання фази `switch-tick`. Якщо цей час менший за тривалість жовтого сигналу `yellow-ticks`, то обидва напрямки показують жовтий.



Код зміни кольорів:

<pre>

;; This procedure checks the variable green-light-up? at each intersection and sets the

;; traffic lights to have the green light up or the green light to the left.

to set-signal-colors  ;; intersection (patch) procedure

&nbsp; 

&nbsp; let yellow-ticks floor (ticks-per-cycle \* yellow-duration / 100)



&nbsp; ;; Розрахунок часу до перемикання

&nbsp; let switch-tick floor ((my-phase \* ticks-per-cycle) / 100)

&nbsp; let ticks-until-switch (switch-tick - phase)

&nbsp; 

&nbsp; if ticks-until-switch < 0 \[ set ticks-until-switch ticks-until-switch + ticks-per-cycle ]



&nbsp; ifelse power?

&nbsp; \[

&nbsp;   ;; Якщо час до зміни менший за тривалість жовтого то вмикаємо жовтий

&nbsp;   ifelse ticks-until-switch < yellow-ticks

&nbsp;   \[

&nbsp;       ask patch-at -1 0 \[ set pcolor yellow ]

&nbsp;       ask patch-at 0 1 \[ set pcolor yellow ]

&nbsp;   ]

&nbsp;   \[

&nbsp;     ifelse green-light-up?

&nbsp;     \[

&nbsp;       ask patch-at -1 0 \[ set pcolor red ]

&nbsp;       ask patch-at 0 1 \[ set pcolor green ]

&nbsp;     ]

&nbsp;     \[

&nbsp;       ask patch-at -1 0 \[ set pcolor green ]

&nbsp;       ask patch-at 0 1 \[ set pcolor red ]

&nbsp;     ]

&nbsp;   ]

&nbsp; ]

&nbsp; \[

&nbsp;   ask patch-at -1 0 \[ set pcolor white ]

&nbsp;   ask patch-at 0 1 \[ set pcolor white ]

&nbsp; ]

end

</pre>



Код зміни сигналів:



<pre>

to set-signals

&nbsp; ask intersections with \[auto?]

&nbsp; \[

&nbsp;   if phase = floor ((my-phase \* ticks-per-cycle) / 100) 

&nbsp;   \[

&nbsp;     set green-light-up? (not green-light-up?)

&nbsp;   ]

&nbsp;   set-signal-colors

&nbsp; ]

end

</pre>



\*\*Додана вірогідність руху на жовтий сигнал.\*\*

Водії тепер бувають нетерплячі і будуть ігнорувати жовтий. Це регулюється новим параметром `impatience`, який вказує на відсоток нетерплячих водіїв в симуляції.

У процедуру `set-car-speed` додано обробку жовтого кольору:

<pre>

&nbsp; ;; жовте світло

&nbsp; if pcolor = yellow \[

&nbsp;   ifelse is-impatient?

&nbsp;   \[ 

&nbsp;     ;; Нетерплячий водій ігнорує жовтий

&nbsp;   ] 

&nbsp;   \[ 

&nbsp;     ;; Чемний водій зупиняється

&nbsp;     set speed 0 

&nbsp;     stop 

&nbsp;   ]

&nbsp; ]

</pre>



Для розуміння які водії нетерплячі в процедурі `set-car-color` було додано фарбування їх машин у помаранчевий:

<pre>

to set-car-color

&nbsp; ...

&nbsp; if is-impatient? \[ set color orange ]

end

</pre>



<br>



\### Внесені зміни у вихідну логіку моделі, на власний розсуд:



\*\*Додано затримку реакції водіїв (Reaction Time).\*\*

У реальному житті водії не рушають миттєво, коли загоряється зелене світло. Додано параметр `reaction-delay` для кожного агента. Коли машина стоїть і світло перемикається на зелене, машина чекає кілька тактів перед початком руху.



Зміни в ініціалізації агентів:

<pre>

turtles-own \[

&nbsp; ...

&nbsp; reaction-timer ;; таймер затримки

]



to setup-cars

&nbsp; ...

&nbsp; set reaction-timer 0

end

</pre>



Зміни в логіці руху `set-car-speed`:

<pre>

to set-car-speed  ;; turtle procedure

&nbsp; if pcolor = red \[ 

&nbsp;   set speed 0

&nbsp;   if reaction-timer = 0 \[ set reaction-timer random 5 ] 

&nbsp;   stop 

&nbsp; ]

&nbsp; ...

&nbsp; ;; Затримка реакції при старті

&nbsp; if speed = 0 \[

&nbsp;   ifelse reaction-timer > 0

&nbsp;   \[

&nbsp;     set reaction-timer reaction-timer - 1

&nbsp;     stop 

&nbsp;   ]

&nbsp;   \[ ]

&nbsp; ]



&nbsp;   ifelse up-car?

&nbsp;   \[ set-speed 0 -1 ]

&nbsp;   \[ set-speed 1 0 ]

&nbsp; 

&nbsp; if speed > 0 \[ set reaction-timer random 5 ]

end

</pre>



!\[Скріншот моделі](Lab2.png)



Фінальний код моделі та її інтерфейс доступні за \[посиланням](model.nlogox).

<br>



\## Обчислювальні експерименти



\### 1. Вплив "нетерплячості" водіїв (рух на жовтий) на пропускну здатність перехрестя

Досліджується залежність середньої швидкості потоку від відсотка водіїв, що намагаються проїхати на жовте світло `impatience`.

Кількість машин фіксована (150), тривалість жовтого сигналу — 10 тактів (при циклі 50). Тривалість однієї симуляції: 500 тактів.



Параметри:

\- \*\*grid-size-x\*\*: 5

\- \*\*grid-size-y\*\*: 5

\- \*\*num-cars\*\*: 150

\- \*\*speed-limit\*\*: 1.0

\- \*\*ticks-per-cycle\*\*: 50

\- \*\*yellow-duration\*\*: 20%

\- \*\*power?\*\*: On



<table>

<thead>

<tr><th>Impatience</th><th>Середня швидкість</th></tr>

</thead>

<tbody>

<tr><td>0%</td><td>0.3</td></tr>

<tr><td>50%</td><td>0.37</td></tr>

<tr><td>100%</td><td>0.25</td></tr>

</tbody>

</table>



!\[Залежність кількості нетерплячих водіїв на середню швидкість](fig1.png)



\*\*Висновок:\*\* Помірна кількість порушників (їзда на жовтий) може трохи збільшити пропускну здатність за рахунок використання "мертвого часу" перемикання, але масова їзда на жовтий призводить до блокування перехресть і зниження загальної швидкості.

