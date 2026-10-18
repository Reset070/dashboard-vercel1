<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>สภาพอากาศปัจจุบัน - จังหวัดกาฬสินธุ์</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Phosphor Icons for beautiful weather icons -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    <style>
        body {
            font-family: 'Prompt', sans-serif;
            /* Default background, will be slightly dynamic via CSS gradient */
            background: linear-gradient(135deg, #74ebd5 0%, #9face6 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0;
            color: #1f2937;
        }

        .glass-card {
            background: rgba(255, 255, 255, 0.25);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.3);
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
        }

        /* Weather specific backgrounds */
        .bg-clear { background: linear-gradient(135deg, #56CCF2 0%, #2F80ED 100%); }
        .bg-cloudy { background: linear-gradient(135deg, #8e9eab 0%, #eef2f3 100%); }
        .bg-rain { background: linear-gradient(135deg, #4b6cb7 0%, #182848 100%); color: white; }
        .bg-thunder { background: linear-gradient(135deg, #141E30 0%, #243B55 100%); color: white; }
        
        /* Smooth transition for background changes */
        body {
            transition: background 1s ease-in-out, color 0.5s ease;
        }

        .skeleton {
            animation: pulse 1.5s infinite;
            background: rgba(255,255,255,0.4);
            border-radius: 0.5rem;
        }

        @keyframes pulse {
            0% { opacity: 0.6; }
            50% { opacity: 0.3; }
            100% { opacity: 0.6; }
        }
    </style>
</head>
<body id="app-body" class="p-4 sm:p-8">

    <div class="glass-card w-full max-w-md rounded-3xl p-8 relative overflow-hidden transition-all duration-300 hover:shadow-2xl">
        <!-- Decoration element -->
        <div class="absolute -top-16 -right-16 w-32 h-32 bg-white opacity-20 rounded-full blur-2xl"></div>
        <div class="absolute -bottom-16 -left-16 w-40 h-40 bg-white opacity-20 rounded-full blur-2xl"></div>

        <!-- Header -->
        <div class="text-center mb-8 relative z-10">
            <h1 class="text-2xl sm:text-3xl font-bold mb-1">จังหวัดกาฬสินธุ์</h1>
            <p class="text-sm opacity-80" id="date-display">กำลังโหลดวันที่...</p>
        </div>

        <!-- Main Weather Info -->
        <div class="flex flex-col items-center justify-center mb-8 relative z-10">
            <div id="weather-icon-container" class="text-7xl mb-4 text-white drop-shadow-md skeleton w-24 h-24 rounded-full flex items-center justify-center">
                <!-- Icon will be injected here -->
            </div>
            
            <div class="flex items-start justify-center">
                <span id="temp-display" class="text-6xl sm:text-7xl font-bold tracking-tighter skeleton w-32 h-16 inline-block"></span>
                <span id="temp-unit" class="text-2xl sm:text-3xl font-medium mt-2 hidden">°C</span>
            </div>
            
            <p id="weather-desc" class="text-xl sm:text-2xl font-medium mt-2 skeleton w-48 h-8"></p>
        </div>

        <!-- Additional Details -->
        <div class="grid grid-cols-2 gap-4 relative z-10">
            <!-- Humidity -->
            <div class="bg-white bg-opacity-30 rounded-2xl p-4 flex flex-col items-center justify-center backdrop-blur-md">
                <i class="ph ph-drop text-2xl mb-1 opacity-80"></i>
                <span class="text-xs font-semibold opacity-70 uppercase tracking-wider mb-1">ความชื้น</span>
                <span id="humidity-display" class="text-lg font-bold skeleton w-12 h-6"></span>
            </div>
            <!-- Wind -->
            <div class="bg-white bg-opacity-30 rounded-2xl p-4 flex flex-col items-center justify-center backdrop-blur-md">
                <i class="ph ph-wind text-2xl mb-1 opacity-80"></i>
                <span class="text-xs font-semibold opacity-70 uppercase tracking-wider mb-1">ความเร็วลม</span>
                <span id="wind-display" class="text-lg font-bold skeleton w-16 h-6"></span>
            </div>
        </div>

        <!-- Footer Note -->
        <div class="text-center mt-6 pt-4 border-t border-white/20 text-xs opacity-80 relative z-10">
            <p class="mb-1">อุณหภูมิปัจจุบันในจังหวัดกาฬสินธุ์ | ข้อมูลจาก Open-Meteo</p>
            <p class="font-semibold text-sm mt-1">จัดทำโดย: นายชนินทร์ , นายวิทวัส CE6941</p>
        </div>
    </div>

    <script>
        // Set current date in Thai format
        const dateOptions = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
        document.getElementById('date-display').textContent = new Date().toLocaleDateString('th-TH', dateOptions);

        // Coordinates for Kalasin, Thailand
        const lat = 16.4333;
        const lon = 103.5000;
        
        // API URL
        const apiUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m&timezone=Asia%2FBangkok`;

        // Weather Code Interpretation (WMO Codes)
        function getWeatherInfo(code) {
            const weatherMap = {
                0: { desc: 'ท้องฟ้าแจ่มใส', icon: 'ph-sun', bg: 'bg-clear' },
                1: { desc: 'แจ่มใสเป็นส่วนมาก', icon: 'ph-sun-dim', bg: 'bg-clear' },
                2: { desc: 'มีเมฆบางส่วน', icon: 'ph-cloud-sun', bg: 'bg-cloudy' },
                3: { desc: 'มีเมฆมาก', icon: 'ph-cloud', bg: 'bg-cloudy' },
                45: { desc: 'มีหมอก', icon: 'ph-cloud-fog', bg: 'bg-cloudy' },
                48: { desc: 'มีหมอกจัด', icon: 'ph-cloud-fog', bg: 'bg-cloudy' },
                51: { desc: 'ฝนปรอยๆ', icon: 'ph-cloud-drizzle', bg: 'bg-cloudy' },
                53: { desc: 'ฝนปรอยๆ', icon: 'ph-cloud-drizzle', bg: 'bg-cloudy' },
                55: { desc: 'ฝนปรอยๆ หนาแน่น', icon: 'ph-cloud-drizzle', bg: 'bg-rain' },
                61: { desc: 'ฝนตกเล็กน้อย', icon: 'ph-cloud-rain', bg: 'bg-rain' },
                63: { desc: 'ฝนตกปานกลาง', icon: 'ph-cloud-rain', bg: 'bg-rain' },
                65: { desc: 'ฝนตกหนัก', icon: 'ph-cloud-rain', bg: 'bg-rain' },
                71: { desc: 'หิมะตก (เป็นไปได้ยาก)', icon: 'ph-snowflake', bg: 'bg-cloudy' },
                73: { desc: 'หิมะตก', icon: 'ph-snowflake', bg: 'bg-cloudy' },
                75: { desc: 'หิมะตกหนัก', icon: 'ph-snowflake', bg: 'bg-cloudy' },
                77: { desc: 'ลูกเห็บตก', icon: 'ph-cloud-snow', bg: 'bg-cloudy' },
                80: { desc: 'ฝนตกเป็นหย่อมๆ', icon: 'ph-cloud-showers-heavy', bg: 'bg-rain' },
                81: { desc: 'ฝนตกหนักเป็นหย่อมๆ', icon: 'ph-cloud-showers-heavy', bg: 'bg-rain' },
                82: { desc: 'ฝนตกหนักมากเป็นหย่อมๆ', icon: 'ph-cloud-showers-heavy', bg: 'bg-rain' },
                95: { desc: 'พายุฝนฟ้าคะนอง', icon: 'ph-cloud-lightning', bg: 'bg-thunder' },
                96: { desc: 'พายุฝนฟ้าคะนองและลูกเห็บ', icon: 'ph-cloud-lightning', bg: 'bg-thunder' },
                99: { desc: 'พายุฝนฟ้าคะนองรุนแรง', icon: 'ph-cloud-lightning', bg: 'bg-thunder' }
            };

            return weatherMap[code] || { desc: 'ไม่ทราบสภาพอากาศ', icon: 'ph-cloud-warning', bg: 'bg-cloudy' };
        }

        async function fetchWeather() {
            try {
                const response = await fetch(apiUrl);
                if (!response.ok) throw new Error('Network response was not ok');
                const data = await response.json();
                
                const current = data.current;
                const weatherInfo = getWeatherInfo(current.weather_code);

                // Remove skeletons
                document.querySelectorAll('.skeleton').forEach(el => {
                    el.classList.remove('skeleton');
                    el.classList.remove('w-32', 'h-16', 'w-48', 'h-8', 'w-12', 'h-6', 'w-16', 'w-24', 'h-24'); // remove sizing classes used for skeleton
                });

                // Update DOM
                document.getElementById('temp-display').textContent = Math.round(current.temperature_2m);
                document.getElementById('temp-unit').classList.remove('hidden');
                
                document.getElementById('weather-desc').textContent = weatherInfo.desc;
                
                document.getElementById('humidity-display').textContent = `${current.relative_humidity_2m}%`;
                document.getElementById('wind-display').textContent = `${current.wind_speed_10m} km/h`;

                // Update Icon
                const iconContainer = document.getElementById('weather-icon-container');
                iconContainer.innerHTML = `<i class="ph ${weatherInfo.icon} text-inherit"></i>`;

                // Update Background based on weather
                document.getElementById('app-body').className = `p-4 sm:p-8 ${weatherInfo.bg}`;

            } catch (error) {
                console.error("Error fetching weather data: ", error);
                document.getElementById('weather-desc').textContent = "ไม่สามารถโหลดข้อมูลได้";
                document.getElementById('weather-desc').classList.remove('skeleton', 'w-48', 'h-8');
            }
        }

        // Initialize
        fetchWeather();
    </script>
</body>
</html>
