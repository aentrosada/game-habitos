<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rastreador de Hábitos</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f9;
        }

        header {
            background-color: #4CAF50;
            color: white;
            text-align: center;
            padding: 1rem;
        }

        .container {
            max-width: 900px;
            margin: 20px auto;
            background: #fff;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
        }

        .login-form, .habit-form {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 20px;
        }

        .login-form input, .habit-form input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .login-form button, .habit-form button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            border-radius: 4px;
        }

        .login-form button:hover, .habit-form button:hover {
            background-color: #45a049;
        }

        .habit-list {
            list-style-type: none;
            padding: 0;
        }

        .habit-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            margin: 5px 0;
            background: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        .habit-item span {
            flex: 1;
        }

        .habit-item button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
            border-radius: 4px;
        }

        .habit-item button:hover {
            background-color: #45a049;
        }

        .medal {
            font-weight: bold;
            color: #FFD700;
        }

        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <header>
        <h1>Rastreador de Hábitos</h1>
        <p>Acompanhe seus hábitos e mantenha a consistência!</p>
    </header>

    <div class="container">
        <!-- Login Section -->
        <form class="login-form" id="login-form">
            <input type="email" id="email" placeholder="Digite seu email" required>
            <input type="password" id="password" placeholder="Digite sua senha" required>
            <button type="submit">Entrar</button>
            <button type="button" id="register-button">Cadastrar</button>
        </form>

        <!-- Habit Tracker Section -->
        <div id="habit-section" class="hidden">
            <form class="habit-form" id="habit-form">
                <input type="text" id="habit-name" placeholder="Adicione um novo hábito" required>
                <button type="submit">Adicionar Hábito</button>
            </form>
            <ul class="habit-list" id="habit-list"></ul>
            <button id="logout-button">Sair</button>
        </div>
    </div>

    <script>
        const loginForm = document.getElementById('login-form');
        const habitSection = document.getElementById('habit-section');
        const habitForm = document.getElementById('habit-form');
        const habitNameInput = document.getElementById('habit-name');
        const habitList = document.getElementById('habit-list');
        const registerButton = document.getElementById('register-button');
        const logoutButton = document.getElementById('logout-button');

        let currentUser = null;
        let users = JSON.parse(localStorage.getItem('users')) || {};

        function getMedal(streak) {
            if (streak >= 90) return '🏆 Platina';
            if (streak >= 75) return '💎 Diamante';
            if (streak >= 60) return '🥇 Ouro III';
            if (streak >= 45) return '🥇 Ouro II';
            if (streak >= 30) return '🥇 Ouro I';
            if (streak >= 21) return '🥈 Prata';
            if (streak >= 15) return '🥉 Bronze I';
            if (streak >= 11) return '🥉 Bronze';
            if (streak >= 3) return '⭐ BETA';
            return '';
        }

        function renderHabits() {
            habitList.innerHTML = '';
            const userHabits = users[currentUser].habits || [];

            userHabits.forEach((habit, index) => {
                const habitItem = document.createElement('li');
                habitItem.className = 'habit-item';

                const habitName = document.createElement('span');
                const medal = getMedal(habit.streak);
                habitName.innerHTML = `${habit.name} - Sequência: ${habit.streak} dias <span class="medal">${medal}</span>`;

                const completeButton = document.createElement('button');
                completeButton.textContent = 'Marcar como Feito';
                completeButton.onclick = () => {
                    habit.streak++;
                    saveUsers();
                    renderHabits();
                };

                const resetButton = document.createElement('button');
                resetButton.textContent = 'Reiniciar Sequência';
                resetButton.style.backgroundColor = '#f44336';
                resetButton.onclick = () => {
                    habit.streak = 0;
                    saveUsers();
                    renderHabits();
                };

                habitItem.appendChild(habitName);
                habitItem.appendChild(completeButton);
                habitItem.appendChild(resetButton);

                habitList.appendChild(habitItem);
            });
        }

        function saveUsers() {
            localStorage.setItem('users', JSON.stringify(users));
        }

        loginForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;

            if (users[email] && users[email].password === password) {
                currentUser = email;
                loginForm.classList.add('hidden');
                habitSection.classList.remove('hidden');
                renderHabits();
            } else {
                alert('Credenciais inválidas ou usuário não encontrado.');
            }
        });

        registerButton.addEventListener('click', () => {
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;

            if (email && password) {
                if (!users[email]) {
                    users[email] = { password, habits: [] };
                    saveUsers();
                    alert('Cadastro realizado com sucesso! Faça login para continuar.');
                } else {
                    alert('Usuário já cadastrado.');
                }
            } else {
                alert('Por favor, preencha email e senha para cadastrar.');
            }
        });

        habitForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const habitName = habitNameInput.value.trim();
            if (habitName) {
                users[currentUser].habits.push({ name: habitName, streak: 0 });
                habitNameInput.value = '';
                saveUsers();
                renderHabits();
            }
        });

        logoutButton.addEventListener('click', () => {
            currentUser = null;
            habitSection.classList.add('hidden');
            loginForm.classList.remove('hidden');
        });
    </script>
</body>
</html>
