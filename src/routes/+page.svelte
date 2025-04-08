<script lang="ts">
    // @ts-nocheck

    import {FilesetResolver, HandLandmarker, HolisticLandmarker} from '@mediapipe/tasks-vision'


    const DENOMINATORS = [200, 160, 125, 100, 80, 63, 50, 40, 32, 25, 20]

    let handLandmarker = undefined
    let video = undefined
    let cameraEnabled = false
    let landoltSymbolPath = ""


    let correct_turns = 0
    let turns = 0

    let line = 0

    const a = 20
    let denominator = DENOMINATORS[line]
    let current_size = denominator/20 * 2.54

    let dir_symbols = {
        "N": {
            "UP": "⭡",
            "DOWN": "⭣",
        },
        "R": {
            "UP": "↗",
            "MID": "⭢",
            "DOWN": "↘"
        },
        "L": {
            "UP": "↖",
            "MID": "⭠",
            "DOWN": "↙"
        }
    }

    function getRandomIntInclusive(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function _pick_random_symbol() {
        
        const verts = ["UP", "DOWN", "MID"]
        const hors = ["N", "R", "L"]

        let hor = hors[getRandomIntInclusive(0, 2)]
        let vert = ""
        if (hor == "N") {
            vert = verts[getRandomIntInclusive(0, 1)]
        } else {
            vert = verts[getRandomIntInclusive(0, 2)]
        }

        landoltSymbolPath = `landolt_c/${hor}_${vert}.webp`

        document.getElementById("landolt_disp").style.height = `${current_size}mm`

        return {hor: hor, vert: vert}
    }

    async function initialize() {

// Hide banner
        document.getElementById("banner").style.display = "none"

        // Configure ML

        const vision = await FilesetResolver.forVisionTasks(
        // path/to/wasm/root
        "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest/wasm"
        );
        handLandmarker = await HandLandmarker.createFromOptions(
            vision,
            {
            baseOptions: {
                modelAssetPath: "https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/latest/hand_landmarker.task"
            },
            numHands: 1,
            minHandDetectionConfidence: 0.85,
            minHandPresenceConfidence: 0.7,
            runningMode: "VIDEO"
            });


             // Get user's camera
        video = document.getElementById("videoOut") as HTMLVideoElement
        // Enable camera

        video.addEventListener("loadeddata", run_inference)


       
        // Request access to the camera
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        // Set the video source to the camera stream
        video.srcObject = stream;
        cameraEnabled = true
    }

    function terminate() {

        // Hide banner
        document.getElementById("banner").style.display = "block"

        video = document.getElementById("videoOut") as HTMLVideoElement
        
        video.srcObject.getTracks().forEach(track => {
            track.stop()
        });


        video.srcObject = undefined
        cameraEnabled = false

        location.reload()
    }

    let disp_symbol = ""
    let lastVideoTime = -1
    let lastTurn = -1
    let newImgLoaded = false

    const step = 0.05
    const timeOut = 0.4

    const timeBeforeSubmission = 4

    let last_hor = ""
    let last_vert = ""
    let time_since_last_change = 0
    let last_recorded_change = -1

    let correct_row = 0

    let correct_direction = undefined

    async function run_inference(event) {
        // console.log("running inference...")

        // Add timeout between turns and captures
        if (!newImgLoaded) {
            correct_direction = _pick_random_symbol()
            // console.log("SHOW SIGN:", correct_direction)
            newImgLoaded = true
        }

        if (Math.abs(lastVideoTime-video.currentTime) > step && (video.currentTime - lastTurn) > timeOut) {
            let res = handLandmarker.detectForVideo(video, video.currentTime)
            // console.log(res)
            lastVideoTime = video.currentTime

            // Infering direction
            let s = [1, 2, 3]
            s.length
            let o_x, o_y, i_x, i_y, cosine, angle, horizontal, diagonal, vertical

            if (res.landmarks.length == 1 ) {
                o_x = res.landmarks[0][5].x
                o_y = res.landmarks[0][5].y

                i_x = res.landmarks[0][8].x
                i_y = res.landmarks[0][8].y

                cosine = (i_y - o_y) / Math.sqrt((i_x - o_x)**2 + (i_y - o_y)**2)
                angle = Math.acos(cosine) / Math.PI * 180
                // console.log(angle / Math.PI * 180)

                // Horizontal direction
                if (o_x < i_x) {
                    horizontal = "L"
                } else {
                    horizontal = "R"
                }

                // console.log(horizontal)

                // Vertical direction
                if (angle <= 22.5) {
                    vertical = "DOWN"
                    horizontal = "N"
                    diagonal = false
                } else if (22.5 < angle && angle <= 67.5) {
                    vertical = "DOWN"
                    diagonal = true
                } else if (67.5 < angle && angle <= 112.5) {
                    vertical = "MID"
                    diagonal = false
                } else if (112.5 < angle && angle <= 157.5) {
                    vertical = "UP"
                    diagonal = true
                } else {
                    diagonal = false
                    vertical = "UP"
                    horizontal = "N"
                }
                console.log(angle)
                console.log(horizontal, vertical, diagonal)
                // disp_symbol = dir_symbols[horizontal][vertical]

                // Update duration since last change
                time_since_last_change = video.currentTime - last_recorded_change
                if ((horizontal != last_hor || vertical != last_vert)) {
                    last_recorded_change = video.currentTime
                    last_hor = horizontal
                    last_vert = vertical
                    
                }

                // Check user direction after timeout
                if (correct_direction.hor == horizontal && correct_direction.vert == vertical) {

                    correct_turns += 1
                    turns += 1

                    if (correct_turns >= 4 && turns == 5) {

                        // If done 4-5 shapes correctly , halve the shape's size
                        correct_direction = _pick_random_symbol()
                        
                        correct_turns = 0
                        turns = 0
                        correct_row += 1
                        line += 1
                        denominator = DENOMINATORS[line]

                        // update size
                        current_size = denominator/20 * 2.54

                        console.log("PASSED ", "20 / ", denominator)

                        if (denominator == 20) {
                            alert(`✅✅ Test completed!\nYour estimated vision acuity is 20/20!`)
                            window.location.reload()
                        }

                    } else if (correct_turns <= 3 && turns == 5) {
                        // If not, terminate the test early
                        alert(`✅ Test completed!\nYour estimated vision acuity is ${Math.round(a/denominator*20)}/20`)
                        window.location.reload()
                    }

                    newImgLoaded = false
                } else {
                    // If not correct after timeout
                    if (time_since_last_change > timeBeforeSubmission) {
                        newImgLoaded = false
                        time_since_last_change = 0
                        last_recorded_change = video.currentTime
                        turns += 1

                        if (correct_turns <= 3 && turns == 5) {
                            // If not, terminate the test early
                            alert(`✅ Test completed!\nYour estimated vision acuity is ${Math.round(a/denominator*20)}/20`)
                            window.location.reload()
                        
                        }

                    }
                }

                time_since_last_change = video.currentTime - last_recorded_change
            } else {
                horizontal = ""
                diagonal = false
                vertical = ""
                time_since_last_change = 0
                last_recorded_change = video.currentTime
            }


        }        

        if (cameraEnabled) {
            window.requestAnimationFrame(run_inference)
        }

    }

    function toggleVideo() {
        if (video.style.display == "none") {
            video.style.display = "block"
        } else {
            video.style.display = "none"
        }
    }

</script>

<style>
    button {
        padding: 10px;
        text-transform: capitalize;
        background-color: rgb(33, 130, 227);
        color: white;
        font-size: 1rem;
        border: solid transparent;
        border-radius: 5px;
    }

    button.secondary {
        background-color: gray;
    }
</style>

<progress value={time_since_last_change} max={timeBeforeSubmission} style="width: 100%; height: 30px"></progress>

<br>

<button on:click={initialize} >Begin Testing</button>
<button on:click={terminate} class="secondary">Stop Testing</button>
<button on:click={toggleVideo} class="secondary">Show/Hide Video</button>
<!-- <button on:click={run_inference}>inference</button> -->
<br>



<div style="display: flex; font-family:system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif">

    <div id="banner" style="">
        <h1>Online vision acuity tester</h1>
        <p>This is a basic, computerized, gesture-based implementation of the <a href="https://en.wikipedia.org/wiki/Landolt_C">Landolt C chart</a>, built with <a href="https://ai.google.dev/edge/mediapipe/solutions/guide">MediaPipe</a> and Svelte. To use this tool:. </p>
        <ol>
            <li>Click on "Begin Testing."</li>
            <li>Stand 1 meter apart from your monitor.</li>
            <li>Point your finger towards the C-shape's gap direction.</li>
        </ol>
        <p style="color:#cd0000"> Testing results should only be treated as references, and might not reflect your actual vision acuity.</p>
        <p>Check out the project's GitHub repo: <a href="https://github.com/nhathuy07/vision-acuity-tester">https://github.com/nhathuy07/vision-acuity-tester</a></p>
    </div>

    <video id="videoOut" autoplay style="padding: 15px; transform: scale(-1, 1); border-radius: 15px;">
        <track kind="captions" default>
    </video>
    <br>
    
    <img src="" alt="">
    <div>
        <img id="landolt_disp" src={landoltSymbolPath} alt="" style="height: 175mm;">
        
        <p style="font-size: 50px"></p>
    </div>
    
</div>

<!-- <p style="font-size: 300px; font-weight: bold; margin: 30px">{time_since_last_change}</p> -->
