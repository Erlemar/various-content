# Введение



Эта статья описывает небольшой самостоятельный проект по распознаванию рукописного ввода цифр. Вы сможете узнать, как сделать сделать следующее практически с нуля следующее:

- создать простой сайт с использованием Flask и Bootstrap;

- разместить его на платформе Heroku;

- реализовать сохранение и загрузку данных на облаке Amazon s3;

- собрать собственный датасет;

- натренировать модели машинного обучения (FNN и CNN);

- сделать возможность дообучения этих моделей;

- объединить всё это в работающий сайт;

Для полного понимания проекта необходимо знать как работает deep learning для распознавания изображений, иметь базовые знания о Flask и немного разбираться в HTML, JS и CSS.



## Немного обо мне

Я лишь около года занимаюсь машинным обучением и всем связанным с этим (до этого 4 года работал в консалтинге на внедрении ERP-систем). Для получения и развития навыков решил делать такие вот проекты, чтобы глубже изучать интересные мне темы, получать практические навыки и в результате получать



## Зарождение идеи

Несколько месяцев назад я прошёл специализацию Яндекса и МФТИ на Coursera. У неё есть свой team в Slack, и в апреле там самоорганизовалась группа для прохождения Стенфордского курса cs231n. Как это происходило - отдельная история, но ближе к делу. Важной частью курса является самостоятельный проект (40% оценки для студентов), в котором предоставляется полная свобода действий. Я не хотел делать что-то серьёзное и потом писать про это статью ибо не видел в этом смысла, но всё же душа просила сделать что-то для достойного завершения курса. Примерно в это мне на глаза попался сайт https://tensorflow-mnist.herokuapp.com/ где можно нарисовать цифру и 2 сетки на Tensorflow мгновенно распознают её и покажут результат.



# Основная часть
Здесь я расскажу о том, что и как я делал, чтобы реализовать проект. Объяснения будут достаточно подробными, чтобы было можно их повторить, но некоторые совсем базовые вещи я буду описывать кратко или пропускать.


## Планирование проекта
Перед тем как приступать к чему-то большому стоит это что-то распланировать. По ходу дела будут выясняться новые подробности и план придётся скорректировать, но некое изначальное видение просто обязано быть.
1. Для любого проекта по машинному обучению одним из основополагающих моментов является вопрос о том, какие данные использовать и где их взять. Датасет MNIST активно используется в задачах распознавация цифр, и именно поэтому я не захотел его использовать. В интернете можно найти примеры подобных проектов, где модели натренированы на MNIST (например, https://github.com/sugyan/tensorflow-mnist), мне же хотелось сделать что-то новое. И, наконец, мне казалось, что многие из цифр в MNIST далеки от реальности - при просмотре датасета я встречал много таких вариантов, которые сложно представить в реальности (если только у человека совсем уж жуткий почерк). Плюс рисование цифр в браузере мышкой несколько отличается от их написания ручкой. Как следствие я решил собрать собственный датасет;
2. Следующий (а точнее одновременный) шаг - это создание сайта для сбора данных, а в дальнейшем - создание модифицирование сайта так, чтобы он мог давать предсказания. На тот момент у меня имелись базовые знания Flask, а также HTML, JS и CSS. Поэтому я и решил делать сайт на Flask, а в качестве хостинга была выбрана платформа Heroku, как позволяющая быстро и просто захостить маленький сайт.
3. Далее предстояло создать сами модели, которые должны делать основную работу. Этот этап казался самым простым, поскольку после cs231n имелся достаточный опыт создания архитектуры нейронных сетей для распознавания изображений. Предварительно хотелось сделать несколько архитектур, но в дальнейшем решил остановиться на двух - FNN и CNN. Кроме того, нужно было сделать возможность дотренировки этих моделей, и некоторые идеи на этот счёт у меня уже были;
4. После подготовки моделей следует придать сайту приличный вид, как-то отображать предсказания, дать возможность оценивать корректность ответа, немного описать функционал и сделать ряд других мелких и не очень вещей. На этапе планирования я не стал уделять много времени на размышления об этом, просто составил список;

## Сбор данных
На сбор данных у меня ушла чуть ли не половина всего времени, потраченного на проект. Дело в том, что я слабо был знаком, с тем, что надо было сделать, поэтому приходилось двигаться методом проб и ошибок.

### Создание первой версии сайта (для сбора данных)
Первый вариант сайта выглядел вот так: 

![](https://raw.githubusercontent.com/Erlemar/various-content/master/1.jpg?raw=true)

В нём была только самая базовая функциональность:

* канвас для рисования;
* радио-кнопки для выбора лейбла;
* кнопки для сохранения картинок и очистки канваса;
* поле, в котором писалось успешно ли было сохранение;
* сохранение картинок на облаке Amazon;

Итак, теперь подробнее обо всём этом. Специально для статьи я сделал минимально рабочую версию сайта, на примере которой и буду рассказывать, как сделать вышеперечисленное: https://digits-little.herokuapp.com/

#### Flask
Flask - питоновский фреймворк для создания сайтов. На официальном сайте есть отличное [введение](http://flask.pocoo.org/docs/0.12/quickstart/#quickstart). Есть разные способы использования Flask для получения и передачи информации, так в этом проекте я использовал AJAX. AJAX даёт возможность "фонового" обмена данными между браузером и веб-сервером, это позволяет не перезагружать страницы каждый раз при передаче данных.

#### Структура проекта

![](https://raw.githubusercontent.com/Erlemar/various-content/master/9.jpg?raw=true)

Все файлы, используемые в проекте можно разделить на 2 неравные группы: меньшая часть необходима для того, чтобы приложение могло работать на Heroku, а все остальные задействованы непосредственно в работе сайта.

#### HTML и JS
HTML-файлы должны храниться в папке "template", на данной стадии было достаточно иметь один.

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8" />
	<title>Handwritten digit recognition</title>
	<link rel="stylesheet" type="text/css" href="static/bootstrap.min.css">
	<link rel="stylesheet" type="text/css" href="static/style.css">
</head>

	<body>

		<div  class="container">
			<div>
				Здесь можно порисовать.<br>
				<canvas id="the_stage" width="200" height="200">fsf</canvas>
				<div>
					<button type="button" class="btn btn-default butt" onclick="clearCanvas()"><strong>clear</strong></button>
					<button type="button" class="btn btn-default butt" id="save" onclick="saveImg()"><strong>save</strong></button>
				</div>

				<div>
					Please select one of the following<br>
					<input type="radio" name="action" value="0" id="digit">0<br>
					<input type="radio" name="action" value="1" id="digit">1<br>
					<input type="radio" name="action" value="2" id="digit">2<br>
					<input type="radio" name="action" value="3" id="digit">3<br>
					<input type="radio" name="action" value="4" id="digit">4<br>
					<input type="radio" name="action" value="5" id="digit">5<br>
					<input type="radio" name="action" value="6" id="digit">6<br>
					<input type="radio" name="action" value="7" id="digit">7<br>
					<input type="radio" name="action" value="8" id="digit">8<br>
					<input type="radio" name="action" value="9" id="digit">9<br>
				</div>
			</div>

			<div class="col-md-6 column">
				<h3>result:</h3>
				<h2 id="rec_result"></h2>
			</div>
		</div>
		<script src="static/jquery.min.js"></script>
		<script src="static/bootstrap.min.js"></script>
		<script src="static/draw.js"></script>
	</body>
</html>
```

В шапке документа и в конце тега body находятся ссылки на файлы js и css. Сами эти файлы должны находиться в папке "static". Теперь подробнее о том, как работает рисование на canvas и сохранение рисунка.
Canvas - это двухмерный элемент HTML5 для рисования. Изображение может рисоваться скриптом, или пользователь может иметь возможность рисовать, используя мышь (или касаясь сенсорного экрана).
Canvas задаётся в HTML следующим образом:
```html
<canvas id="the_stage" width="200" height="200"> </canvas>
```

До этого я не был знаком с этим элементом HTML, поэтому мои изначальные попытки сделать возможность рисования были неудачными. Через некоторое время я нашёл работающий пример и позаимствовал его (ссылка есть в моём файле draw.js).

```javascript
var drawing = false;
var context;
var offset_left = 0;
var offset_top = 0;

function start_canvas () {
    var canvas = document.getElementById ("the_stage");
    context = canvas.getContext ("2d");
    canvas.onmousedown = function (event) {mousedown(event)};
    canvas.onmousemove = function (event) {mousemove(event)};
    canvas.onmouseup   = function (event) {mouseup(event)};
    for (var o = canvas; o ; o = o.offsetParent) {
    offset_left += (o.offsetLeft - o.scrollLeft);
    offset_top  += (o.offsetTop - o.scrollTop);
    }
    draw();
}

function getPosition(evt) {
    evt = (evt) ?  evt : ((event) ? event : null);
    var left = 0;
    var top = 0;
    var canvas = document.getElementById("the_stage");

    if (evt.pageX) {
    left = evt.pageX;
    top  = evt.pageY;
    } else if (document.documentElement.scrollLeft) {
    left = evt.clientX + document.documentElement.scrollLeft;
    top  = evt.clientY + document.documentElement.scrollTop;
    } else  {
    left = evt.clientX + document.body.scrollLeft;
    top  = evt.clientY + document.body.scrollTop;
    }
    left -= offset_left;
    top -= offset_top;

    return {x : left, y : top}; 
}

function
mousedown(event) {
    drawing = true;
    var location = getPosition(event);
    context.lineWidth = 8.0;
    context.strokeStyle="#000000";
    context.beginPath();
    context.moveTo(location.x,location.y);
}

function
mousemove(event) {
    if (!drawing) 
        return;
    var location = getPosition(event);
    context.lineTo(location.x,location.y);
    context.stroke();
}

function
mouseup(event) {
    if (!drawing) 
        return;
    mousemove(event);
	context.closePath();
    drawing = false;
}

.
.
.
onload = start_canvas;
```
При загрузке страницы сразу запускается функция start_canvas. Первые две строчки находят канвас как элемент с определенным id ("the stage") и определяют его как двухмерное изображение.
При рисовании на canvas есть 3 события: onmousedown, onmousemove и onmouseup. Есть ещё аналогичные события для касаний, но об этом позже.

onmousedown - происходит при клике на canvas. В этот момент задается ширина и цвет линии, а также определяется начальная точка рисования. На словах определение местоположения курсора звучит просто, но по факту это не совсем тривиально. Для нахождения точки используется функция **getPosition()** - она находит координаты курсора на странице и определяет координаты точки на canvas с учетом относительного положения canvas на странице и с учетом того, что страница может быть проскроллена. После нахождения точки **context.beginPath()** начинает путь рисования, а **context.moveTo(location.x,location.y)** "передвигает" этот путь к точке, которая была определена в момент клика.

onmousemove - следование за движением мышки при нажатой левой клавише. В самом начале сделана проверка на то, что клавиша нажата (то есть drawing = true), если же нет - рисование не осуществляется. **context.lineTo()** создаёт линию по траектории движения мыши, а **context.stroke()** непосредственно рисует её.

mouseup - происходит при отпускании левой клавиши мыши. **context.closePath()** завершает рисование линии.

Вот так и осуществляется рисование на canvas. В интерфейсе есть ещё 4 элемента:

* "Поле" с текущим статусом. JS обращается к нему по id (rec_result) и отображает текущий статус. Статус либо пуст, либо показывает, что изображение сохранено, либо показывает название сохранённого изображения.
```html
            <div class="col-md-6 column">
                <h3>result:</h3>
                <h2 id="rec_result"></h2>
            </div>
```

* Радио-кнопки для выбора цифры. На этапе сбора данных нарисованным цифрам нужно как-то присваивать лейблы, для этого и были добавлены 10 кнопок. Кнопки задаются одинаковым способом: `<input type="radio" name="action" value="0" id="digit">0<br>`, где на месте 0 стоит соответствующая цифра. *Name* используется для того, чтобы JS мог получить значение активной радио-кнопки (*value*);
* Кнопка для очищения canvas - чтобы можно было нарисовать новую цифру. `<button type="button" class="btn btn-default butt" id="save" onclick="saveImg()"><strong>save</strong></button>` При нажатии на эту кнопку происходит следующее:

```javascript
function draw() {
    context.fillStyle = '#ffffff';
    context.fillRect(0, 0, 200, 200);
}

function clearCanvas() {
    context.clearRect (0, 0, 200, 200);
    draw();
    document.getElementById("rec_result").innerHTML = "";
}
```

Содержимое canvas очищается, и он заливается белым цветом. Также статус становится пустым.

* Наконец, кнопка сохранения нарисованного изображения. Она вызывает следующую функцию Javascript:

```javascript
	function saveImg() {
	document.getElementById("rec_result").innerHTML = "connecting...";
	var canvas = document.getElementById("the_stage");
	var dataURL = canvas.toDataURL('image/jpg');
	var dig = document.querySelector('input[name="action"]:checked').value;
	$.ajax({
	  type: "POST",
	  url: "/hook",
	  data:{
		imageBase64: dataURL,
		digit: dig
		}
	}).done(function(response) {
	  console.log(response)
	  document.getElementById("rec_result").innerHTML = response
	});
	
}
```
Сразу же после нажатия кнопки в поле статуса отбражается значение "connecting...". Затем изображение конвертируется в текстовую строку с помощью метода кодирования base64. Результат выглядит следующим образом: `"data:image/png;base64,%string%`, где можно увидеть тип файла (image), расширение (png), кодирование base64 и сам стринг. Тут хочу заметить, что я слишком поздно заметил ошибку в моём коде. Мне следовало использовать 'image/jpeg' как аргумент для `canvas.toDataURL()`, но я сделал опечатку и в итоге изображения по факту были png.
Далее я беру значение активной радио-кнопки (по name='action' и по состоянию `checked`) и сохраняю в переменную `dig`.

Наконец, AJAX запрос отправляет закодированное изображение и лейбл в питон, а затем получает ответ. Я довольно много времени потратил на то, чтобы заставить работать эту конструкцию, постараюсь объяснить, что происходит в каждой строке.
Вначале указывается тип запроса - в данном случае "POST", то есть данные из JS передаются в python скрипт.
"/hook" - это куда передаются данные. Поскольку я использую Flask, то я могу в нужном декораторе указать "/hook" в качестве URL, и это будет означать, что именно функция в этом декораторе будет использоваться, когда запрос POST идут на этот URL. Подробнее об этом в разделе про Flask ниже.
data - это данные, которые передаются в запросе. Вариантов передачи данных много, я задаю значение и имя через которое можно получить это значение.
Наконец, `done()` - это то, что происходит при успешном выполнении запроса. Мой AJAX запрос возвращает некий ответ (а точнее текст с именем сохраненного изображения), этот ответ вначале выводится в консоль (для отладки), а затем отображается в поле статуса.

#### Flask и сохранение изображения
Теперь перейдём уже к тому, как данные из AJAX запроса попадают в python, и как изображение сохраняется.
Основной скрипт - **main.py**.

```python
__author__ = 'Artgor'
from functions import Model
from flask import Flask, render_template, request
from flask_cors import CORS, cross_origin
import base64
import os

app = Flask(__name__)
model = Model()
CORS(app, headers=['Content-Type'])

@app.route("/", methods=["POST", "GET", 'OPTIONS'])
def index_page(text="", prediction_message=""):

	return render_template('index.html', text=text, prediction_message=prediction_message)

@app.route('/hook', methods = ["GET", "POST", 'OPTIONS'])
def get_image():
	if request.method == 'POST':
		image_b64 = request.values['imageBase64']
		drawn_digit = request.values['digit']
		print('Data received')
		image_encoded = image_b64.split(',')[1]
		image = base64.decodebytes(image_encoded.encode('utf-8'))		
		save = model.save_image(drawn_digit, image)	

		print('Done')
	return save

if __name__ == '__main__':
	port = int(os.environ.get("PORT", 5000))
	app.run(host='0.0.0.0', port=port, debug=False)
```



### Интеграция с Amazon s3



### Heroku



## FNN

Тренировка



## Bootstrap и дизайн сайта



## Дообучение FNN и модификации дизайна под это



## CNN

Тренировка

Дообучение



## Всякие улучшения.

Touch, MNIST, чистка кода (и облом с переменными в tf). Страницы с описанием проекта.



# Запуск проекта



## Слежение за точность.



# Заключение



## Существующие проблемы



## Планы на следующий проект



## Общее впечатление от проекта
