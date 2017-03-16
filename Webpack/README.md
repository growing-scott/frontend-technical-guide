# Webpack
Webpack은 bundler(배포)와 개발 서버 등을 지원하며, Webpack 2.x 버전 기준으로 작성하였습니다.

- SPA Application 개발 또는 의존 관계를 가지는 프로젝트에서 root 모듈을 기준으로 트리 구조(hierarchy)를 가지고 있다.
    ```
        아래와 같은 디렉토리 구조를 가지는 application을 배포하는 단계에서는 하나의 파일(bundle)로 만들어서 배포한다.
        Webpack은 대표적으로 진입점에서 시작하여 각 모듈간 의존관계를 추적하여 하나의 파일로 묶어주는 역할을 한다.
        - root.js(entry - 진입점)
            - a.js
                - a-1.js
                - a-2.js
            - b.js
                - b-b-1.js
                    - b-b-b-1.js
    ```    


- webpack.config.js 파일을 가지며, 일반적으로 배포와 개발용으로 나눠서 관리한다.
- 개발: dev-webpack.config.js
    - 개발서버(hot module) 설정
    - entry, loader 설정
- 배포: prod-webpack.config.js
    - output 설정
    - loader 설정
    - plugins
        - minify
        - less to css
        - image to metadata
        - code splitting
        - tree shaking



## Concepts

1. Entry
    - root 모듈의 시작점. 프로젝트의 진입점. js, less 등이 포함된다.
    - string, array, object 형태를 지원하며, 각각의 특징이 있다.
    - string
        - 단일 진입점으로 구성되는 프로젝트의 경우에 해당되며, output도 하나의 파일만 생성된다.

        ```
         {
             entry: './src/index.js',
             output: {
                 path: '/dist',
                 filename: 'bundle.js'
             }
         }
        ```

    - array
        - 단일 진입점이나 덪붙여서 생성될 필요가 있는 경우에 해당되며, output은 하나의 파일이 생성된다.
        - attach1, attach2는 index.js와 의존성이 없으며, index.js로 생성된 bundle.js에 attach1, attach2가 추가된다.
        - 실제 application에서 의존관계가 없는 파일을 추가하고자 할 때 사용한다.

        ```
         {
             entry: ['./src/index.js', '/src/attach1.js', '/src/attach2.js'],
             output: {
                 path: '/dist',
                 filename: 'bundle.js'
             }
         }
        ```

    - object
        - 다중 페이지 Application을 개발 중이고, 각각 진입점(a, b, c index page)가 있다면 해당되는 경우이다.
        - output도 하나의 bundle.js가 아닌 key(명칭)으로 bundle파일이 생성된다.

        ```
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
        ```

2. Output
    - bundle 파일이나 plugin(less, css 등)을 이용한 배포에 최종 생성 파일 정보를 설정하는 부분
    ```
    {
        output: {
            path: '/build',     // 파일 저장 경로
            publicPath: 'http://localhost:8080' // 배포 빌드시에 prefix개념으로 css, html등에서 사용되는 url 업데이트 경로
            filename: 'out.js'  // 저장될 파일명
        }
    }
    ```

    - publicPath 지정이 필수는 아니며, image, css의 url에 영향을 준다.
    ```
        // url loader 예시

        // index.css
        .image{
            background-image: url('./test.png');
        }

        // webpack
        output: {
            publicPath: 'http://localhost'
        },
        plugin: {
            {
                test: /\.png$/,
                loader: "url-loader"
            }
        }

        // 최종 output에는
        .image{
            background-image: url('http://localhost/test.png');
        }
    ```


3. Loaders
    - 현재의 브라우저 허용 포멧에 맞게 변환해주는 역활을 하며, js, less, css등을 지원한다.
    - 자바스크립트의 경우 ES6로 작성한 파일을 babel-loader를 통하여 현재 지원 브라우저에 맞게 변환된 js를 만들어준다.
    - less로 작성된 경우에도 css-loader를 통하여 css로 생성해준다.
    - loader에는 단일 loader와 chaining loader로 구분된다.
    - chaining loader는 여러 loader를 연결하여 실행하며, !로 구분하며, 오른쪽에서 왼쪽 방향으로 실행된다.


    ```
    // 단일 loader 사용
    module: {
        loaders: [{
            test: /\.js$/,              // 정규표현식으로 표현하며, js파일에 대하여 babel-loader를 적용한다는 의미
            exclude: /node_modules/,    // 제외 폴더
            loader: ‘babel-loader’      // babel loader 사용
        }]
    }

    // chaining loader 사용
    module: {
        loaders: [{
            test: /\.css$/,              // css 파일만 대상
            exclude: /node_modules/,    // 제외 폴더
            loader: 'style-loader!css-loader'     // css 파일을 해석하여, style 태그안에 삽입하는 loader
        }]
    }
    /* 위 예제 기준 chaining 동작 방식
        ex) main.css 파일을 가지고 index.html 파일안에 <style>Main CSS Content</style>을 삽입하고 싶다면..
        1. Webpack은 모듈안에 의존적인 CSS파일들을 검색
        2. webpack은 require(main.css’)을 가지고 있는지 JS파일을 체크하고 찾으면, 먼저 해당 파일에 대해 css-loader 수행한다.
        3. css-loader는 모든 CSS와 그안에 의존적인 다른 CSS(ex: import otherCSS) 를 JSON파일로 로드한다.
        4. 그후 Webpack은 결과들을 style-loader로 보낸다.
        5. style-loader로 JSON을 가져와서, <style>CSS content</style> tag 를 추가하고, index.html 파일안에 tag를 삽입한다.
    */

    ```

4. Plugins
