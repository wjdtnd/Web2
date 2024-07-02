시작이미지
![시작이미지](https://github.com/wjdtnd/Web2/assets/162400276/e39e8dd6-b1d4-40c7-8a51-657cdc7648dd)


#코드

html 코드이다.

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Movie Recommendation</title>
    <style>
        h1 { color: white; background-color: black; }
        h2 { color: white; background-color: red; }
        .card { display: inline-block; margin: 10px; }
        .card-img-top { width: 150px; height: 150px; }
    </style>
</head>
<body>
<h1>영화 추천</h1>

    <h2>전체 영화</h2>
    <div id="alldiv"></div>
    <button type="button" onclick="MovieObject.getall()">전체 영화 로드</button>

    <br><br><br>

    <h2>장르별 추천</h2>
    <select id="keyword">
        <option value="none">=== 선택 ===</option>
        <option value="comedy">코미디</option>
        <option value="romance">로맨스</option>
        <option value="sci-fi">SF</option>
        <option value="thriller">스릴러</option>
        <option value="adventure">어드벤쳐</option>
        <option value="family">가족</option>
        <option value="crime">범죄물</option>
        <option value="fantasy">판타지</option>
        <option value="drama">드라마</option>
        <option value="animation">애니메이션</option>
        <option value="mystery">미스테리</option>
        <option value="musical">뮤지컬</option>
        <option value="horror">호러</option>
        <option value="action">액션</option>
        <option value="war">전쟁</option>
        <option value="western">웨스턴</option>
        <option value="documentary">다큐멘터리</option>        
    </select>
    <button type="button" onclick="MovieObject.getgenres()">장르 선택 완료</button>

    <br><br><br>

    <h2>영화 기반 추천</h2>
    <form>
        영화1: <input id='movie1' type='text'/><br>
        영화2: <input id='movie2' type='text'/><br>
        영화3: <input id='movie3' type='text'/><br>
        <button type="button">입력 완료</button>
    </form>

    <div id="result"></div>

    <script src="/static/movieScript.js"></script>
</body>
</html>


자바스크립트 코드이다

    
    let MovieObject = {
    getall: function() {
        let movielist = [
            { poster_path: "https://via.placeholder.com/150?text=Image1" },
            { poster_path: "https://via.placeholder.com/150?text=Image2" },
            { poster_path: "https://via.placeholder.com/150?text=Image3" },
            { poster_path: "https://via.placeholder.com/150?text=Image4" },
            { poster_path: "https://via.placeholder.com/150?text=Image5" },
            { poster_path: "https://via.placeholder.com/150?text=Image6" },
            { poster_path: "https://via.placeholder.com/150?text=Image7" },
            { poster_path: "https://via.placeholder.com/150?text=Image8" },
            { poster_path: "https://via.placeholder.com/150?text=Image9" },
            { poster_path: "https://via.placeholder.com/150?text=Image10" }
        ];

        let topdiv = document.getElementById("alldiv");

        // Clear previous images
        topdiv.innerHTML = '';

        for (let i = 0; i < movielist.length; i++) {
            let cmovie = document.createElement('div');
            cmovie.className = 'card';

            let mimg = document.createElement('img');
            mimg.className = 'card-img-top';
            mimg.src = movielist[i].poster_path;

            cmovie.appendChild(mimg);
            topdiv.appendChild(cmovie);
        }
    },

    getgenres: function() {
        const genre = document.getElementById('keyword').value;
        if (genre !== 'none') {
            fetch(`http://127.0.0.1:8000/genres/${genre}`)
                .then(response => response.json())
                .then(data => {
                    console.log(data.result);
                    const resultDiv = document.getElementById('result');
                    resultDiv.innerHTML = '';
                    data.result.forEach(movie => {
                        const movieElement = document.createElement('div');
                        movieElement.textContent = movie;
                        resultDiv.appendChild(movieElement);
                    });
                })
                .catch(error => console.error('Error:', error));
        }
    }
    };

    MovieObject.getall();


파이썬 코드이다


    from fastapi import FastAPI, Query, Request
    from fastapi.responses import HTMLResponse
    from fastapi.staticfiles import StaticFiles
    from fastapi.templating import Jinja2Templates
    from starlette.middleware.cors import CORSMiddleware
    from typing import Optional, List
    import uvicorn
    import pandas as pd


    from fastapi.staticfiles import StaticFiles

    app = FastAPI()

    app.mount("/static", StaticFiles(directory="static"), name="static")

    df = pd.read_excel('movie.xlsx')


   from app.recommender import item_based_recommendation, user_based_recommendation
   from app.resolver import random_items, random_genre_items

   app = FastAPI()


  origins = ["*"]
  app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    )


    templates = Jinja2Templates(directory="templates")


    app.mount("/static", StaticFiles(directory="static"), name="static")

    @app.get("/", response_class=HTMLResponse)
    async def read_root(request: Request):
        return templates.TemplateResponse("index.html", {"request": request})

    @app.get("/all/")
     async def random_movies():
     result = random_items()
     return {"result": result}

    @app.get("/genres/{genre}")
     async def genre_movies(genre: str):
     result = random_genre_items(genre)
     return {"result": result}

   @app.get("/item-based/{item_id}")
     async def item_based(item_id: str):
     result = item_based_recommendation(item_id)
     return {"result": result}

  @app.get("/user-based/")
    async def user_based(params: Optional[List[str]] = Query(None)):
        input_ratings_dict = dict(
            (int(x.split(":")[0]), float(x.split(":")[1])) for x in params)
        print(input_ratings_dict)
        result = user_based_recommendation(input_ratings_dict)
        return {"result": result}

   @app.get("/weather/")
async def weather(q: Optional[List[str]] = Query(None), units: str = 'metric'):
    print(q)
    print(units)
    q_dict = dict((int(x.split(":")[0]), float(x.split(":")[1])) for x in q)
    print(q_dict)
    return {"result": f'q={q} : units={units}'}

if __name__ == '__main__':
    uvicorn.run(app, host='127.0.0.1', port=8000)



