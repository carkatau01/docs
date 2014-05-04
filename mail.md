git f53f291f67507108dd0f5772a7eb33bc6458d840

---

# Работа с e-mail

- [Настройка](#configuration)
- [Основы использования](#basic-usage)
- [Добавление встроенных вложений](#embedding-inline-attachments)
- [Очереди отправки](#queueing-mail)
- [Локальная разработка](#mail-and-local-development)

<a name="configuration"></a>
## Настройка

Laravel предоставляет простой интерфейс к популярной библиотеке [SwiftMailer](http://swiftmailer.org). Главный файл настроек - `app/config/mail.php` - содержит всевозможные параметры, позволяющие вам менять SMTP-сервер, порт, логин, пароль, а также устанавливать глобальный адрес `from` для исходящих сообщений. Вы можете использовать любой SMTP-сервер, либо стандартную функцию PHP `mail` - для этого установите параметр `driver` в значение `mail`. Кроме того, доступен драйвер `sendmail`.

<a name="basic-usage"></a>
## Основы использования

Метод `Mail::send` используется для отправки сообщения:

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'Джон Смит')->subject('Привет!');
	});

Первый параметр - имя шаблона, который должен использоваться для текста сообщения. Второй - массив переменных, передаваемых в шаблон. Третий - функция-замыкание, позволяющая вам внести дополнительные настройки в сообщение.

> **Примечание:**  переменная `$message` всегда передаётся в ваш шаблон и позволяет вам прикреплять вложения. Таким образом, вам не стоит передавать одноимённую переменную в массиве `$data`.

В дополнение к шаблону в формате HTML вы можете указать текстовый шаблон письма:

	Mail::send(array('html.view', 'text.view'), $data, $callback);

Вы также можете оставить только один формат, передав массив с ключом `html` или `text`:

	Mail::send(array('text' => 'view'), $data, $callback);

Вы можете указывать другие настройки для сообщения, например, копии или вложения:

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});

При добавлении файлов можно указывать их MIME-тип и/или отображаемое имя:

	$message->attach($pathToFile, array('as' => $display, 'mime' => $mime));

> **Примечание:** Объект $message, передаваемый функции-замыканию метода `Mail::send`, наследует класс собщения SwiftMailer, что позволяет вам вызывать любые методы для создания своего сообщения.

<a name="embedding-inline-attachments"></a>
## Добавление встроенных вложений

Обычно добавление встроенных вложений в письмо обычно утомительное занятие, однако Laravel делает его проще, позволяя вам добавлять файлы и получать соответствующие CID.

> Встроенные (inline) вложения - файлы, не видимые получателю в списке вложений, но используемые внутри HTML-тела сообщения; CID - уникальный идентификатор внутри данного сообщения, используемый вместо URL в таких атрибутах, как `src` - прим. пер.

#### Добавление картинки в шаблон сообщения

	<body>
		Вот какая-то картинка:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### Добавление встроенной в html картинки (data:image)

	<body>
		А вот картинка, полученная из строки с данными:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

Переменная `$message` всегда передаётся шаблонам сообщений классом `Mail`.

<a name="queueing-mail"></a>
## Очереди отправки

Из-за того, что отправка множества мейлов может сильно повлиять на время отклика приложения, многие разработчики помещают их в очередь на отправку. Laravel позволяет делать это, используя [единое API очередей](/docs/queues). Для помещения сообщения в очередь просто используйте метод `Mail::queue()`:

#### Помещение сообщения в очередь отправки

	Mail::queue('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'Джон Смит')->subject('Привет!');
	});

Вы можете задержать отправку сообщения на нужное число секунд методом `later`:

	Mail::later(5, 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'Джон Смит')->subject('Привет!');
	});

Если же вы хотите поместить сообщение в определённую очередь отправки, то используйте методы  `queueOn` и `laterOn`:

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'Джон Смит')->subject('Привет!');
	});

<a name="mail-and-local-development"></a>
## Локальная разработка

При разработке приложения обычно предпочтительно отключить доставку отправляемых сообщений. Для этого вы можете либо вызывать метод `Mail::pretend`, либо установить параметр `pretend` в значение `true` в файле настроек `app/config/mail.php`. Когда это сделано, сообщения будут записываться в файл журнала вашего приложения, вместо того, чтобы быть отправленными получателю.

#### Включение симуляции отправки

	Mail::pretend();