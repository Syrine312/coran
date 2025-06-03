
<html lang="fr">
<head>
    <meta charset="UTF-8" />
   
    <style>
        :root {
            --primary-color: #4a6fa5;
            --secondary-color: #ffc0cb;
            --accent-color: #ff8c94;
            --light-color: #f8f9fa;
            --dark-color: #343a40;
            --border-radius: 8px;
            --box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
            background-color: #f5f7fa;
            color: var(--dark-color);
            line-height: 1.6;
        }

        h2 {
            color: var(--primary-color);
            margin-bottom: 25px;
            text-align: center;
            font-size: 2rem;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
        }

        canvas {
            border: 2px solid var(--primary-color);
            border-radius: var(--border-radius);
            cursor: crosshair;
            margin-top: 20px;
            box-shadow: var(--box-shadow);
            background-color: white;
            max-width: 100%;
        }

        .controls {
            display: flex;
            gap: 15px;
            align-items: center;
            margin: 20px 0;
            flex-wrap: wrap;
            justify-content: center;
            background-color: white;
            padding: 15px;
            border-radius: var(--border-radius);
            box-shadow: var(--box-shadow);
            width: 90%;
            max-width: 800px;
        }

        label {
            display: flex;
            flex-direction: column;
            align-items: center;
            font-weight: 500;
            color: var(--primary-color);
            gap: 5px;
        }

        input[type="color"] {
            width: 40px;
            height: 40px;
            border: 2px solid var(--primary-color);
            border-radius: 50%;
            cursor: pointer;
            padding: 0;
        }

        input[type="color"]::-webkit-color-swatch {
            border: none;
            border-radius: 50%;
        }

        input[type="range"] {
            width: 100px;
            cursor: pointer;
        }

        button {
            padding: 10px 20px;
            font-size: 14px;
            cursor: pointer;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: var(--border-radius);
            transition: all 0.3s ease;
            font-weight: 500;
            box-shadow: var(--box-shadow);
        }

        button:hover {
            background-color: var(--accent-color);
            transform: translateY(-2px);
        }

        button:active {
            transform: translateY(0);
        }

        #upload {
            display: none;
        }

        .upload-btn {
            padding: 12px 24px;
            background-color: var(--primary-color);
            color: white;
            border-radius: var(--border-radius);
            cursor: pointer;
            font-weight: 500;
            margin-bottom: 15px;
            box-shadow: var(--box-shadow);
            transition: all 0.3s ease;
        }

        .upload-btn:hover {
            background-color: var(--accent-color);
        }

        #clear {
            background-color: #dc3545;
        }

        #clear:hover {
            background-color: #c82333;
        }

        #download {
            background-color: #28a745;
        }

        #download:hover {
            background-color: #218838;
        }

        @media (max-width: 768px) {
            .controls {
                flex-direction: column;
                align-items: stretch;
            }
            
            label {
                flex-direction: row;
                justify-content: space-between;
                width: 100%;
            }
        }
    </style>
</head>

<body>
    <h2>تشجيع على حفظ القران</h2>

    <label for="upload" class="upload-btn">Choisir une image</label>
    <input type="file" id="upload" accept="image/*" />

    <div class="controls">
        <label>
            Couleur :
            <input type="color" id="colorPicker" value="#ffc0cb" />
        </label>
        <label>
            Opacité :
            <input
                type="range"
                id="opacitySlider"
                min="0"
                max="1"
                step="0.01"
                value="0.2"
            />
        </label>
        <label>
            Taille pinceau :
            <input type="range" id="brushSize" min="1" max="100" value="15" />
        </label>
        <button id="clear">Effacer tout</button>
        <button id="download">Télécharger l'image</button>
    </div>

    <canvas id="canvas"></canvas>

    <script>
        const upload = document.getElementById("upload");
        const canvas = document.getElementById("canvas");
        const ctx = canvas.getContext("2d");

        const colorPicker = document.getElementById("colorPicker");
        const opacitySlider = document.getElementById("opacitySlider");
        const brushSize = document.getElementById("brushSize");
        const clearBtn = document.getElementById("clear");
        const downloadBtn = document.getElementById("download");

        let drawing = false;
        let img = new Image();

        // Charger image sauvegardée
        const savedImage = localStorage.getItem("savedCanvas");
        if (savedImage) {
            const tempImg = new Image();
            tempImg.onload = function () {
                canvas.width = tempImg.width;
                canvas.height = tempImg.height;
                ctx.drawImage(tempImg, 0, 0);
            };
            tempImg.src = savedImage;
        }

        // Chargement image
        upload.addEventListener("change", (e) => {
            const file = e.target.files[0];
            const reader = new FileReader();

            reader.onload = function (event) {
                img = new Image();
                img.onload = function () {
                    canvas.width = img.width;
                    canvas.height = img.height;
                    ctx.drawImage(img, 0, 0);
                    saveCanvas(); // Sauvegarde auto après chargement image
                };
                img.src = event.target.result;
            };

            if (file) {
                reader.readAsDataURL(file);
            }
        });

        // Sauvegarde automatique
        function saveCanvas() {
            const dataURL = canvas.toDataURL("image/png");
            localStorage.setItem("savedCanvas", dataURL);
        }

        // Dessin
        canvas.addEventListener("mousedown", () => (drawing = true));
        canvas.addEventListener("mouseup", () => {
            drawing = false;
            saveCanvas(); // Sauvegarde après dessin
        });
        canvas.addEventListener("mouseleave", () => {
            drawing = false;
            saveCanvas();
        });
        canvas.addEventListener("mousemove", draw);

        function draw(e) {
            if (!drawing) return;

            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;

            const color = colorPicker.value;
            const opacity = parseFloat(opacitySlider.value);
            const size = parseInt(brushSize.value);

            const r = parseInt(color.slice(1, 3), 16);
            const g = parseInt(color.slice(3, 5), 16);
            const b = parseInt(color.slice(5, 7), 16);

            ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${opacity})`;
            ctx.beginPath();
            ctx.arc(x, y, size, 0, Math.PI * 2);
            ctx.fill();
        }

        // Effacer tout
        clearBtn.addEventListener("click", () => {
            if (img.src) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.drawImage(img, 0, 0);
            } else {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            }
            saveCanvas(); // Sauvegarde après effacement
        });

        // Télécharger l'image finale
        downloadBtn.addEventListener("click", () => {
            const link = document.createElement("a");
            link.download = "image_coloriee.png";
            link.href = canvas.toDataURL("image/png");
            link.click();
        });
    </script>
</body>
</html>
