<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Al-Tuba Restaurant Billing System</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&family=Open+Sans:wght@300&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Roboto', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f9f9f9;
            text-align: center;
            color: #333;
        }
        .container {
            width: 90%;
            max-width: 700px;
            margin: 20px auto;
            padding: 25px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
            animation: fadeIn 1s ease;
        }
        h1 {
            color: #ff5733;
            font-size: 32px;
            font-weight: 700;
            margin-bottom: 30px;
        }
        h2 {
            color: #f39c12;
        }
        .menu-item {
            display: flex;
            justify-content: space-between;
            padding: 15px;
            margin: 10px 0;
            background-color: #f8f8f8;
            border-radius: 8px;
            transition: all 0.3s ease;
        }
        .menu-item:hover {
            background-color: #ffedd2;
            transform: scale(1.02);
        }
        .menu-item span {
            font-weight: bold;
        }
        button {
            padding: 10px 15px;
            background-color: #ff5733;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        button:hover {
            background-color: #d44b1f;
        }
        .order-summary {
            background-color: #fff3cd;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .order-summary p {
            font-size: 16px;
            margin: 5px 0;
        }
        .order-summary h3 {
            margin-bottom: 15px;
            font-size: 24px;
            color: #ff5733;
        }
        .order-list {
            list-style: none;
            padding-left: 0;
            margin-bottom: 15px;
        }
        .input-field {
            padding: 12px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            width: 80%;
            max-width: 300px;
        }
        .payment-btn {
            width: 100%;
            margin-top: 20px;
            background-color: green;
            padding: 12px;
            border: none;
            border-radius: 5px;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }
        .payment-btn:hover {
            background-color: darkgreen;
        }
        .dark-mode-btn {
            margin: 15px 0;
            background-color: #333;
            color: white;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            border-radius: 5px;
        }
        .dark-mode-btn:hover {
            background-color: #555;
        }
        @media screen and (max-width: 600px) {
            .container {
                width: 100%;
                padding: 15px;
            }
            button, .payment-btn {
                width: 100%;
            }
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Al-Tuba Restaurant</h1>
    <button class="dark-mode-btn" onclick="toggleDarkMode()">Toggle Dark Mode</button>

    <!-- Login Section -->
    <div id="login-section" class="login-section">
        <h3>Please Log In</h3>
        <input type="text" id="username" class="input-field" placeholder="Username" required>
        <input type="password" id="password" class="input-field" placeholder="Password" required>
        <button onclick="login()">Login</button>
    </div>

    <!-- Menu Section -->
    <div id="menu" class="menu" style="display: none;"></div>

    <!-- Order Summary Section -->
    <div id="order-summary" class="order-summary" style="display: none;">
        <h3>Order Summary</h3>
        <ul id="order-list" class="order-list"></ul>
        <p>Subtotal: $<span id="subtotal">0.00</span></p>
        <p>Tax (10%): $<span id="tax">0.00</span></p>
        <p>Discount: $<span id="discount">0.00</span></p>
        <p>Total: $<span id="total">0.00</span></p>
        <button class="payment-btn" onclick="processPayment()">Process Payment</button>
    </div>
</div>

<script>
    // Sample menu items
    const menuItems = [
        { name: "Burger", price: 5.99, category: "Main" },
        { name: "Pizza", price: 8.99, category: "Main" },
        { name: "Pasta", price: 7.49, category: "Main" },
        { name: "Fries", price: 3.49, category: "Side" },
        { name: "Soda", price: 1.99, category: "Beverage" },
        { name: "Water", price: 0.99, category: "Beverage" }
    ];

    let order = [];
    let loggedIn = false;

    function renderMenu() {
        const menuContainer = document.getElementById("menu");
        menuContainer.innerHTML = "";
        menuItems.forEach((item, index) => {
            const div = document.createElement("div");
            div.classList.add("menu-item");
            div.innerHTML = `
                <span>${item.name} - $${item.price.toFixed(2)}</span>
                <button onclick="addToOrder(${index})">Add</button>
            `;
            menuContainer.appendChild(div);
        });
        menuContainer.style.display = "block"; // Show menu after login
    }

    function addToOrder(index) {
        let existingItem = order.find(o => o.name === menuItems[index].name);
        if (existingItem) {
            existingItem.quantity++;
        } else {
            order.push({ ...menuItems[index], quantity: 1 });
        }
        updateOrderSummary();
    }

    function updateOrderSummary() {
        const orderList = document.getElementById("order-list");
        orderList.innerHTML = "";
        let subtotal = 0;

        order.forEach((item, i) => {
            subtotal += item.price * item.quantity;
            const li = document.createElement("li");
            li.innerHTML = `
                ${item.name} (x${item.quantity}) - $${(item.price * item.quantity).toFixed(2)}
                <button onclick="increaseQuantity(${i})">+</button>
                <button onclick="decreaseQuantity(${i})">-</button>
                <button onclick="removeItem(${i})">X</button>
            `;
            orderList.appendChild(li);
        });

        const tax = subtotal * 0.10;
        const discount = subtotal >= 20 ? 5 : 0;
        const total = subtotal + tax - discount;

        document.getElementById("subtotal").innerText = subtotal.toFixed(2);
        document.getElementById("tax").innerText = tax.toFixed(2
