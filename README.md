<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Toycambo - លក់ប្រដាប់ក្មេងលេង និងកាំភ្លើងទឹក</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Bayon&family=Koulen&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #2563eb;
            --secondary: #f59e0b;
            --dark: #1e293b;
            --light: #f8fafc;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Koulen', 'Bayon', sans-serif;
            background-color: var(--light);
            color: var(--dark);
            overflow-x: hidden;
        }

        /* Header with animated gradient */
        header {
            background: linear-gradient(135deg, #1e3a8a 0%, #2563eb 100%);
            color: white;
            text-align: center;
            padding: 3rem 1rem;
            position: relative;
            overflow: hidden;
        }

        header h1 {
            font-size: 3.5rem;
            margin-bottom: 1rem;
        }

        /* Navigation */
        nav {
            background-color: var(--dark);
            padding: 1rem;
            position: sticky;
            top: 0;
            z-index: 100;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        nav ul {
            display: flex;
            justify-content: center;
            list-style: none;
            gap: 2rem;
        }

        nav a {
            color: white;
            text-decoration: none;
            font-size: 1.2rem;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            transition: all 0.3s ease;
        }

        nav a:hover {
            color: var(--secondary);
            transform: translateY(-2px);
        }

        /* Main Content */
        .content {
            max-width: 1200px;
            margin: 2rem auto;
            padding: 2rem;
            background-color: white;
            border-radius: 12px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
        }

        .product-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 2rem;
        }

        .product-card {
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            overflow: hidden;
            text-align: center;
            transition: all 0.3s ease;
            position: relative;
        }

        .product-card img {
            height: 200px;
            width: 100%;
            object-fit: contain;
        }

        .product-info {
            padding: 1rem;
        }

        .product-price {
            font-size: 1.2rem;
            color: var(--primary);
            font-weight: bold;
            margin-bottom: 1rem;
        }

        .btn {
            display: inline-block;
            background: var(--primary);
            color: white;
            padding: 0.6rem 1.5rem;
            border-radius: 6px;
            text-decoration: none;
            font-size: 1rem;
            transition: all 0.3s ease;
            border: none;
            cursor: pointer;
        }

        .btn:hover {
            background: #1d4ed8;
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(37, 99, 235, 0.3);
        }

        /* Cart */
        .cart-container {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 300px;
            background-color: white;
            padding: 20px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
            border-radius: 8px;
            display: none;
        }

        .cart-container.show {
            display: block;
        }

        .cart-item {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }

        .cart-item span {
            font-weight: bold;
        }

        /* Invoice */
        .invoice {
            margin-top: 2rem;
            padding: 1rem;
            background-color: var(--light);
            border-radius: 8px;
        }
    </style>
</head>
<body>

    <!-- Header Section -->
    <header>
        <h1>Toycambo</h1>
        <p>លក់ប្រដាប់ក្មេងលេង និងកាំភ្លើងទឹក</p>
    </header>

    <!-- Navigation Bar -->
    <nav>
        <ul>
            <li><a href="#home">ទំព័រដើម</a></li>
            <li><a href="#products">ផលិតផល</a></li>
            <li><a href="#contact">ទំនាក់ទំនង</a></li>
        </ul>
    </nav>

    <!-- Main Content Section -->
    <section id="products">
        <h2>ផលិតផលពេញនិយម</h2>
        <div class="product-grid">
            <!-- Product 1 -->
            <div class="product-card" id="product-1">
                <img src="https://assets.onecompiler.app/43bqg7kby/43bqgbkha/1.png" alt="Product Image">
                <div class="product-info">
                    <h3>Product 1</h3>
                    <p>Description goes here</p>
                    <div class="product-price">$20.00</div>
                    <button class="btn add-to-cart" data-product="1">Add to Cart</button>
                </div>
            </div>

            <!-- Product 2 -->
            <div class="product-card" id="product-2">
                <img src="https://assets.onecompiler.app/43bqg7kby/43bqgbkha/9.png" alt="Product Image">
                <div class="product-info">
                    <h3>Product 2</h3>
                    <p>Description goes here</p>
                    <div class="product-price">$25.00</div>
                    <button class="btn add-to-cart" data-product="2">Add to Cart</button>
                </div>
            </div>
        </div>
    </section>

    <!-- Cart Modal -->
    <div class="cart-container" id="cart-container">
        <h3>Your Cart</h3>
        <div id="cart-items"></div>
        <div id="cart-total"></div>
        <button class="btn" onclick="generateInvoice()">Generate Invoice</button>
    </div>

    <!-- Footer Section -->
    <footer>
        <p>&copy; 2024 Toycambo. All rights reserved.</p>
    </footer>

    <script>
        let cart = [];

        // Add to Cart Functionality
        document.querySelectorAll('.add-to-cart').forEach(button => {
            button.addEventListener('click', (e) => {
                const productId = e.target.getAttribute('data-product');
                const product = {
                    id: productId,
                    name: `Product ${productId}`,
                    price: productId === '1' ? 20 : 25, // Adjust prices
                    quantity: 1
                };

                // Check if the product is already in the cart
                const existingProduct = cart.find(item => item.id === productId);
                if (existingProduct) {
                    existingProduct.quantity++;
                } else {
                    cart.push(product);
                }

                updateCart();
            });
        });

        // Update Cart Display
        function updateCart() {
            const cartContainer = document.getElementById('cart-container');
            const cartItems = document.getElementById('cart-items');
            const cartTotal = document.getElementById('cart-total');

            cartItems.innerHTML = '';
            let total = 0;

            cart.forEach(item => {
                total += item.price * item.quantity;
                cartItems.innerHTML += `
                    <div class="cart-item">
                        <span>${item.name} x ${item.quantity}</span>
                        <span>$${item.price * item.quantity}</span>
                    </div>
                `;
            });

            cartTotal.innerHTML = `<strong>Total: $${total}</strong>`;
            cartContainer.classList.add('show');
        }

        // Generate Invoice
        function generateInvoice() {
            let invoiceContent = '<h3>Invoice</h3>';
            cart.forEach(item => {
                invoiceContent += `
                    <p>${item.name} x ${item.quantity} = $${item.price * item.quantity}</p>
                `;
            });

            const total = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
            invoiceContent += `<p><strong>Total: $${total}</strong></p>`;

            const invoiceDiv = document.createElement('div');
            invoiceDiv.classList.add('invoice');
            invoiceDiv.innerHTML = invoiceContent;
            document.body.appendChild(invoiceDiv);

            // Reset Cart after invoice generation
            cart = [];
            updateCart();
        }
    </script>
</body>
</html>
