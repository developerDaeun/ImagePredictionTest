# ImagePredictionTest
TeachableMachine-React-Test

# Image Test

## Phone & iPad

- Model URL
    
    [https://teachablemachine.withgoogle.com/models/0BEUR8k_M/](https://teachablemachine.withgoogle.com/models/0BEUR8k_M/)
    
- Tensorflow.js 코드
    
    ```jsx
    <div>Teachable Machine Image Model</div>
    <button type="button" onclick="init()">Start</button>
    <div id="webcam-container"></div>
    <div id="label-container"></div>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>
    <script type="text/javascript">
        // More API functions here:
        // https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image
    
        // the link to your model provided by Teachable Machine export panel
        const URL = "https://teachablemachine.withgoogle.com/models/0BEUR8k_M/";
    
        let model, webcam, labelContainer, maxPredictions;
    
        // Load the image model and setup the webcam
        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";
    
            // load the model and metadata
            // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
            // or files from your local hard drive
            // Note: the pose library adds "tmImage" object to your window (window.tmImage)
            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();
    
            // Convenience function to setup a webcam
            const flip = true; // whether to flip the webcam
            webcam = new tmImage.Webcam(200, 200, flip); // width, height, flip
            await webcam.setup(); // request access to the webcam
            await webcam.play();
            window.requestAnimationFrame(loop);
    
            // append elements to the DOM
            document.getElementById("webcam-container").appendChild(webcam.canvas);
            labelContainer = document.getElementById("label-container");
            for (let i = 0; i < maxPredictions; i++) { // and class labels
                labelContainer.appendChild(document.createElement("div"));
            }
        }
    
        async function loop() {
            webcam.update(); // update the webcam frame
            await predict();
            window.requestAnimationFrame(loop);
        }
    
        // run the webcam image through the image model
        async function predict() {
            // predict can take in an image, video or canvas html element
            const prediction = await model.predict(webcam.canvas);
            for (let i = 0; i < maxPredictions; i++) {
                const classPrediction =
                    prediction[i].className + ": " + prediction[i].probability.toFixed(2);
                labelContainer.childNodes[i].innerHTML = classPrediction;
            }
        }
    </script>
    ```
    

## 프로젝트 순서

1. npm 설치
    
    ```java
    npm i
    npm install @sashido/teachablemachine-node
    npm install express
    ```
    
2. index.js
    - 코드
        
        ```java
        const express = require("express");
        const TeachableMachine = require("@sashido/teachablemachine-node");
        const bodyParser = require("body-parser");
        
        const model = new TeachableMachine({
          modelUrl: "https://teachablemachine.withgoogle.com/models/0BEUR8k_M/",
        });
        
        const app = express();
        app.use(express.json());
        app.use(bodyParser.json());
        const port = process.env.port || 5000;
        
        app.use(express.urlencoded({ extended: false }));
        
        app.get("/", (req, res) => {
          res.send(`
            <form action="/image/classify" method="POST">
            <p>
            Enter Image Url</p>
            <input name='ImageUrl' autocomplete=off>
            <button>Predict Image</button>
            </form>
            `);
        });
        
        app.post("/image/classify", async (req, res) => {
          const url = req.body.ImageUrl;
        
          return model
            .classify({
              imageUrl: url,
            })
            .then((predictions) => {
              console.log(predictions);
              res.json(predictions);
            })
            .catch((e) => {
              console.error(e);
              res.status(500).send("Something went wrong!");
            });
        });
        
        app.listen(port, () => {
          console.log(`Example app listening at http://localhost:${port}`);
        });
        ```
        
3. 실행
    
    ```java
    npm start
    ```
    
4. 실행 결과
    - iPad 테스트 이미지 URL
        
        ```java
        data:image/jpeg;base64,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
        ```
        
    - 실행 결과
    ![image](https://user-images.githubusercontent.com/38871826/153037898-e32ea313-2cef-4306-b46a-554f1be08165.png)
    ![image](https://user-images.githubusercontent.com/38871826/153037943-81eb13ce-e1d6-473d-b2d3-33a087a87b97.png)

