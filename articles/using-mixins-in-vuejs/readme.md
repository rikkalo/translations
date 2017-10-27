# Используем миксины во Vue.js

*Перевод статьи [Sarah Drasner](https://css-tricks.com/author/sdrasner/): [Using Mixins in Vue.js](https://css-tricks.com/using-mixins-vue-js/)*

![](https://cdn-images-1.medium.com/max/1000/1*JAbXlOdwiVhwfXla9zE7Nw.png)

Как это обычно бывает: у нас есть два во многом похожих компонента, имеющих одинаковую базовую функциональность, но достаточно появления минимального отличия, чтобы встал вопрос, делать два разных компонента или же сделать один компонент, меняя необходимые параметры через `props`?

Ни одно из решений не является идеальным: если вы примете решение разделить на два компонента, то рискуете обновлять его в двух местах в случае изменения логики в будущем. К тому же это решение нарушит принцип DRY (не повторяйся).

С другой стороны, слишком много `props` приведет к тому, что даже автор кода без погружения в контекст не поймет, как это работает.

Познакомьтесь с миксинами. Миксины (*mixins* или примеси) во Vue.js полезны для написания кода в функциональном стиле, потому что, в конечном итоге, функциональное программирование - это создание понятного кода за счёт сокращения изменяемых частей. Миксины позволяют вам инкапсулировать одну функциональность, чтобы вы могли использовать её в разных компонентах. Если миксины написаны правильно, то они не изменяют ничего вне своей области действия: вы всегда будете получать одинаковую логику в разных местах использования. Это и правда очень удобно.

## Базовый пример

Допустим у нас есть два различных компонента, задача которых заключается в переключении видимости: модальное окно и тултип.

Эти компоненты не имеют между собой ничего общего кроме одинаковой логики:

![](https://cdn-images-1.medium.com/max/1000/1*B3Mbc8-x1GxwtxDg5XcLIQ.png)

Мы могли бы извлечь логику и создать миксин, который можно использовать повторно:

![](https://cdn-images-1.medium.com/max/1000/1*m0SoV-mwTg_LMHmr50HPZQ.png)

https://codepen.io/sdras/pen/101a5d737b31591e5801c60b666013db

Разумеется, использование миксинов применимо к куда большему количеству ситуаций. Этот пример намеренно сделан простым и компактным для удобства чтения.

## Использование

Предыдущий пример не показывает как использовать миксины в реальном приложении, поэтому давайте посмотрим на следующий.

Вы можете настроить структуру проекта любым удобным для вас способом, я же создаю папку `mixins`, чтобы всё было организованно.

Файл имеет расширение `.js` в отличие от наших обычных `.vue` компонентов. Экспортируем наш миксин:

![](https://cdn-images-1.medium.com/max/1000/1*lCXeQ51slInBPiDfi4pYrg.png)

Затем импортируем его в `modal.vuе` вот так:

![](https://cdn-images-1.medium.com/max/800/1*MyYBEm3-9lYnZx8aPnqXow.png)

Важно понимать, что несмотря на то, что мы используем объект, а не компонент, хуки жизненного цикла всё ещё доступны для нас.

## Слияние

Рассматривая последний пример, мы можем увидеть, что у нас есть не только логика, но и хуки жизненного цикла, доступные нам из миксина.

Другой вопрос, а как поведут себя хуки при наложении, то есть если мы создадим хук `created` в миксине и в компоненте?

По умолчанию, сначала применяются методы и хуки из миксина, затем из компонента, это сделано для того чтобы при необходимости мы могли переопределить логику миксина.

Это действительно имеет значение: когда возникает конфликт, за компонентом последнее слово.

![](https://cdn-images-1.medium.com/max/800/1*YE4ijLZrLiCjGvuVA3GA3w.png)

При конфликте, мы увидим, кто победит:

![](https://cdn-images-1.medium.com/max/800/1*dsgKb0WoTCPwAbpBvokcqQ.png)

Вы могли заметить, что у нас два вывода `console.log` для экземпляра Vue вместо одного, это потому что первый вызванный метод не был уничтожен: он был переопределен. Мы все ещё вызываем метод `sayHello()`.

## Глобальные миксины

Когда мы используем слово «Глобальные» в отношении миксинов, мы не имеем ввиду возможность доступа к ним в каждом компоненте, ведь мы уже можем получить к ним доступ, импортируя там, где нужно.

Глобальные миксины буквально применяются к каждому компоненту, по этой причине использовать их нужно с осторожностью.

Использование, которое я могу предположить, это что-то вроде плагина, где вам может понадобится доступ ко всем компонентам. Но опять же, даже в этом случае я был бы очень осторожен, особенно когда вы расширяете логику для приложений, которые писали не вы.

Чтобы создать глобальный миксин, нужно разместить его над Vue экземпляром. В обычной vue-cli сборке это будет в файле `main.js`.

![](https://cdn-images-1.medium.com/max/1000/1*XzMhbRqDkIbQ2-N_HwYA1Q.png)

Используйте с осторожностью! Теперь `console.log` появится в каждом компоненте, в этом случае это не так плохо, за исключением кучи визуального мусора в консоли, зато вы можете увидеть насколько потенциально опасно это может быть.

## Заключение

Миксины могут быть полезны для инкапсуляции различной небольшой логики, которую вы хотите повторно использовать.

Это не единственный вариант для решения проблемы, однако мне нравятся миксины потому что можно абстрагироваться от данных, чего например не получится в случае с `props`.

Однако решение нужно применять в зависимости от ситуации, что наиболее логично подходит в контексте вашей задачи.
 - - -

*Читайте нас на [Медиуме](https://medium.com/devschacht), контрибьютьте на [Гитхабе](https://github.com/devSchacht), общайтесь в [группе Телеграма](https://t.me/devSchacht), следите в [Твиттере](https://twitter.com/DevSchacht) и [канале Телеграма](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht)

[Статья на Medium]()