<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เก็บและลบข้อมูลใน IndexedDB พร้อมอนิเมชั่น</title>
    <style>
        /* ทำให้หน้าตาของเว็บดูเรียบง่ายและทันสมัย */
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f4f7fc;
            margin: 0;
            padding: 0;
            transition: background-color 0.5s ease;
        }

        header {
            background-color: #4CAF50;
            color: white;
            text-align: center;
            padding: 1rem 0;
        }

        h2 {
            margin: 0;
        }

        .container {
            width: 90%;
            max-width: 600px;
            margin: 2rem auto;
            padding: 2rem;
            background-color: white;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            transition: transform 0.3s ease;
        }

        .container:hover {
            transform: scale(1.02);
        }

        input[type="text"],
        input[type="email"],
        button {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
            transition: background-color 0.3s ease;
        }

        button {
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
            border: none;
            transition: background-color 0.3s ease;
        }

        button:hover {
            background-color: #45a049;
        }

        ul {
            list-style-type: none;
            padding: 0;
            margin-top: 1rem;
        }

        li {
            padding: 10px;
            border-bottom: 1px solid #ddd;
            display: flex;
            justify-content: space-between;
            align-items: center;
            opacity: 0;
            animation: fadeIn 0.5s forwards;
        }

        /* ปรับขนาดปุ่มลบให้เล็กลง */
        li button {
            background-color: #f44336;
            color: white;
            border: none;
            padding: 4px 10px; /* ปรับขนาดของ padding ให้น้อยลง */
            font-size: 12px; /* ขนาดฟอนต์ที่เล็กลง */
            cursor: pointer;
            border-radius: 4px;
            transition: background-color 0.3s ease;
        }

        li button:hover {
            background-color: #d32f2f;
        }

        .message {
            text-align: center;
            margin-top: 1rem;
            font-weight: bold;
            color: #333;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
            }
            to {
                opacity: 1;
            }
        }

        .message.success {
            color: #4CAF50;
        }

        .message.error {
            color: #f44336;
        }

    </style>
</head>
<body>
    <header>
        <h2>โปรดกรอกข้อมูล</h2>
    </header>

    <div class="container">
        <label for="name">อีเมล:</label>
        <input type="text" id="name" name="name" placeholder="กรุณากรอกชื่อของคุณ">

        <label for="email">รหัสผ่าน:</label>
        <input type="email" id="email" name="email" placeholder="กรุณากรอกอีเมลของคุณ">

        <button onclick="saveData()">บันทึกข้อมูล</button>
        <div class="message" id="message"></div>

        <h3>ข้อมูลที่เก็บไว้:</h3>
        <ul id="dataList"></ul>
    </div>

    <script>
        let db;

        // สร้างหรือเปิดฐานข้อมูล
        const request = indexedDB.open('userDatabase', 1);

        // สร้าง object store เมื่อฐานข้อมูลยังไม่มี
        request.onupgradeneeded = function(event) {
            db = event.target.result;
            if (!db.objectStoreNames.contains('users')) {
                const objectStore = db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
                objectStore.createIndex('name', 'name', { unique: false });
                objectStore.createIndex('email', 'email', { unique: true });
            }
        };

        // เมื่อฐานข้อมูลเปิดสำเร็จ
        request.onsuccess = function(event) {
            db = event.target.result;
            console.log('ฐานข้อมูลเปิดสำเร็จ');
            displayData();
        };

        // เมื่อการเปิดฐานข้อมูลล้มเหลว
        request.onerror = function(event) {
            console.error('การเปิดฐานข้อมูลล้มเหลว:', event.target.error);
        };

        // ฟังก์ชันบันทึกข้อมูล
        function saveData() {
            const name = document.getElementById('name').value;
            const email = document.getElementById('email').value;

            if (!name || !email) {
                document.getElementById('message').innerText = 'กรุณากรอกข้อมูลให้ครบถ้วน!';
                document.getElementById('message').classList.add('error');
                return;
            }

            const transaction = db.transaction(['users'], 'readwrite');
            const objectStore = transaction.objectStore('users');

            const newUser = { name: name, email: email };

            const request = objectStore.add(newUser);

            request.onsuccess = function() {
                document.getElementById('message').innerText = 'ข้อมูลถูกบันทึกเรียบร้อย!';
                document.getElementById('message').classList.add('success');
                displayData();
            };

            request.onerror = function(event) {
                document.getElementById('message').innerText = 'การบันทึกข้อมูลล้มเหลว';
                document.getElementById('message').classList.add('error');
            };
        }

        // ฟังก์ชันแสดงข้อมูล
        function displayData() {
            const objectStore = db.transaction('users').objectStore('users');
            const dataList = document.getElementById('dataList');
            dataList.innerHTML = '';

            const request = objectStore.getAll();
            request.onsuccess = function(event) {
                const users = event.target.result;
                users.forEach(user => {
                    const listItem = document.createElement('li');
                    listItem.innerHTML = `อีเมล: ${user.name}, รหัสผ่าน: ${user.email}`;

                    const deleteButton = document.createElement('button');
                    deleteButton.textContent = 'Delete';
                    deleteButton.onclick = function() {
                        deleteUser(user.id);
                    };
                    listItem.appendChild(deleteButton);

                    dataList.appendChild(listItem);
                });
            };
        }

        // ฟังก์ชันลบข้อมูลตาม id
        function deleteUser(id) {
            const transaction = db.transaction(['users'], 'readwrite');
            const objectStore = transaction.objectStore('users');
            const request = objectStore.delete(id);

            request.onsuccess = function() {
                document.getElementById('message').innerText = 'ข้อมูลถูกลบเรียบร้อย!';
                document.getElementById('message').classList.add('success');
                displayData();
            };

            request.onerror = function(event) {
                document.getElementById('message').innerText = 'การลบข้อมูลล้มเหลว';
                document.getElementById('message').classList.add('error');
            };
        }
    </script>
</body>
</html>
