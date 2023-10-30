Основной задачей вебпака является анализ модулей, их опциональное преобразование и интеллектуальное объединение в один или более бандл (на который можно сослаться в index.html), поэтому вебпаку нужно знать три вещи:
1.	Точка входа приложения
      Сколько бы модулей не содержало приложение, всегда имеется единственная точка входа. Этот модуль включает в себя остальные. Обычно, таким файлом является index.js:
      index.js
      imports about.js
      imports dashboard.js
      imports graph.js
      imports auth.js
      imports api.js

Если мы сообщим вебпаку путь до этого файла, он использует его для создания графа зависимостей приложения. Для этого необходимо добавить свойство entry в настройки вебпака со значением пути к главному файлу:
module.exports = {
entry: './src/index.js'
}
2.	Преобразования, которые необходимо выполнить
      По умолчанию при создании графика зависимостей на основе операторов import / require() вебпак способен обрабатывать только JavaScript и JSON-файлы. Основной задачей лоадеров, как следует из их названия, является предоставление вебпаку возможности работать не только с JS и JSON-файлами.
      Первым делом нужно установить лоадер. Поскольку мы хотим загружать SVG, с помощью npm устанавливаем svg-loader, для стилей используется css-loader, мы хотим не только импортировать такие файлы, но и поместить их в тег <style>, чтобы они применялись к элементам DOM, для этого нужен style-loader
      npm i svg-inline-loader -D
      npm i css-loader -D
      npm i style-loader -D
      npm i babel-loader -D

Далее добавляем его в настройки вебпака. Все лоадеры включаются в массив объектов module.rules. Информация о лоадере состоит из двух частей. Первая — тип обрабатываемых файлов (.svg в нашем случае). Вторая — лоадер, используемый для обработки данного типа файлов (svg-inline-loader в нашем случае). Поскольку для обработки CSS-файлов используется два лоадера, значением свойства use является массив, важен порядок следования лоадеров, сначала style-loader, затем css-loader. Вебпак будет применять их в обратном порядке. Сначала он использует css-loader для импорта './styles.css', затем style-loader для внедрения стилей в DOM. Лоадеры могут использоваться не только для импорта файлов, но и для их преобразования, наприклад babel-loader.
module.exports = {
entry: './src/index.js',
module: {
rules: [
{ test: /\.svg$/, use: 'svg-inline-loader' },
{ test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
{ test: /\.(js)$/, use: 'babel-loader' }
    ]
}
}
3.	Место, в которое следует поместить сформированный бандл
      Следующим шагом является указание директории для бандла. Для этого нужно добавить свойство output в настройки вебпака. Весь процесс выглядит примерно так:
1.	Вебпак получает точку входа, находящуюся в ./app/index.js
2.	Он анализирует операторы import / require и создает граф зависимостей
3.	Вебпак начинает собирать бандл, преобразовывая код с помощью соответствующих лоадеров
4.	Он собирает бандл и помещает его в dist/index_bundle.js
      const path = require('path')

module.exports = {
entry: './src/index.js',
module: {
rules: [
{ test: /\.svg$/, use: 'svg-inline-loader' },
{ test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
{ test: /\.(js)$/, use: 'babel-loader' }
]
},
output: {
path: path.resolve(__dirname, 'dist'),
filename: 'index_bundle.js'
}
}
Плагины (plugins)
В отличие от лоадеров, плагины позволяют выполнять задачи после сборки бандла, вы можете думать о плагинах как о более мощных, менее ограниченных лоадерах.
HtmlWebpackPlugin
Далее добавляем его в настройки вебпака. Все лоадеры включаются в массив объектов module.rules. Информация о лоадере состоит из двух частей. Первая — тип обрабатываемых файлов (.svg в нашем случае). Вторая — лоадер, используемый для обработки данного типа файлов (svg-inline-loader в нашем случае). Поскольку для обработки CSS-файлов используется два лоадера, значением свойства use является массив, index.html

npm i html-webpack-plugin -D

в настройках вебпака:

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
entry: './app/index.js',
module: {
rules: [
{ test: /\.svg$/, use: 'svg-inline-loader' },
{ test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
{ test: /\.(js)$/, use: 'babel-loader' }
]
},
output: {
path: path.resolve(__dirname, 'dist'),
filename: 'index_bundle.js'
},
plugins: [
new HtmlWebpackPlugin()
]
}
( после установки mode в значение production этот плагин не нужен EnvironmentPlugin )
Плагин является частью вебпака, так что его не нужно устанавливать, он позволит React осуществить сборку в режиме продакшна, удалив инструменты разработки, такие как предупреждения. Теперь в любом месте нашего приложения мы можем установить режим продакшна с помощью process.env.NODE_ENV.

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')

module.exports = {
entry: './app/index.js',
module: {
rules: [
{ test: /\.svg$/, use: 'svg-inline-loader' },
{ test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
{ test: /\.(js)$/, use: 'babel-loader' }
]
},
output: {
path: path.resolve(__dirname, 'dist'),
filename: 'index_bundle.js'
},
plugins: [
new HtmlWebpackPlugin(),
new webpack.EnvironmentPlugin({
'NODE_ENV': 'production'
})
]
}
Режим (mode)
В процессе подготовки приложения к продакшну, необходимо выполнить несколько действий, например минификацию кода и удаление комментариев для уменьшения размера бандла. Существуют специальные плагины для решения указанных задачи, но есть более легкий способ. В настройках вебпака можно установить mode в значение development или production в зависимости от среды разработки. После установки mode в значение production вебпак автоматически присваивает process.env.NODE_ENV значение production.

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
entry: './app/index.js',
module: {
rules: [
{ test: /\.svg$/, use: 'svg-inline-loader' },
{ test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
{ test: /\.(js)$/, use: 'babel-loader' }
]
},
output: {
path: path.resolve(__dirname, 'dist'),
filename: 'index_bundle.js'
},
plugins: [
new HtmlWebpackPlugin()
],
mode: 'production'
}
Запуск вебпака
У нас есть файл package.json, в котором мы можем создать script для запуска webpack. Теперь при выполнении команды npm run build в терминале будет запущен вебпак, который создаст оптимизированный бандл index_bundle.js и поместит его в dist.

"scripts": {
"build": "webpack"
}
Режимы разработки и продакшна, Сервер для разработки
Для переключения между режимами необходимо создать два скрипта в package.json, npm run build будет собирать продакшн-бандл, npm run start будет запускать сервер для разработки и следить за изменениями файлов.
Вебпак-сервер для разработки, вместо создания дирекории dist, он хранит данные в памяти и обрабатывает их на локальном сервере, более того, он поддерживает живую перезагрузку

npm i webpack-dev-server -D

"scripts": {
"build": "SET NODE_ENV='production' && webpack",
"start": "webpack-dev-server"
},
Теперь в настроках вебпака мы можем менять значение mode в зависимости от process.env.NODE_ENV.

mode: process.env.NODE_ENV === 'production' ? 'production' : 'development'
