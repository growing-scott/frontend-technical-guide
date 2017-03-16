# Webpack
Webpack 2.x 버전 기준으로 작성하였습니다.
Webpack은 bundler(배포)와 개발 서버 등을 지원한다.

webpack.config.js 파일을 가지며, 일반적으로 배포와 개발용으로 나눠서 관리한다.
개발: dev-webpack.config.js
    - 개발서버(hot module) 설정
    - entry, loader 설정
배포: prod-webpack.config.js
    - output 설정
    - loader 설정
    - plugins
        - minify
        - less to css
        - image to metadata
        - code splitting



## Concepts

1. Entry
    -root 모듈의 시작점. 프로젝트의 진입점. js, less 등이 포함된다.
    - string, array, object 형태를 지원하며, 각각의 특징이 있다.
    - string
        - 단일 진입점으로 구성되는 프로젝트의 경우에 해당되며, output도 하나의 파일만 생성된다.
        `
         {
             entry: './src/index.js',
             output: {
                 path: '/dist',
                 filename: 'bundle.js'
             }
         }
        `

    - array
        - 단일 진입점이나 덪붙여서 생성될 필요가 있는 경우에 해당되며, output은 하나의 파일이 생성된다.
        - attach1, attach2는 index.js와 의존성이 없으며, index.js로 생성된 bundle.js에 attach1, attach2가 추가된다.
        - 실제 application에서 의존관계가 없는 파일을 추가하고자 할 때 사용한다.
        `
         {
             entry: ['./src/index.js', '/src/attach1.js', '/src/attach2.js'],
             output: {
                 path: '/dist',
                 filename: 'bundle.js'
             }
         }
        `

    - object
        - 다중 페이지 Application을 개발 중이고, 각각 진입점(a, b, c index page)가 있다면 해당되는 경우이다.
        - output도 하나의 bundle.js가 아닌 key(명칭)으로 bundle파일이 생성된다.
        `
         {
             entry: {
                 'home': './src/home/index.js',
                 'work': './src/work/index.js'
                 'vendor': ['jquery', 'utils.js', 'attach.js'] // Object - Array형태도 지원하며, CommonsChunkPlugin 참조
                 .......
             },
             output: {
                 path: '/dist',
                 filename: '[name].js'  // out files - home.js, work.js, vendor.js
             }
         }
        `    

2. Output

3. Loaders

4. Plugins
