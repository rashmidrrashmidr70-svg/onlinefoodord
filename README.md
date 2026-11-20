# onlinefoodord
An Online Food Ordering System is a convenient web and mobile platform that lets customers browse restaurant menus, customize dishes, and pay securely online. It simplifies ordering with live delivery tracking and automated kitchen updates, offering diners fast doorstep service while helping restaurants boost sales and minimize o
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CraveCart - Online Food Ordering System</title>
  <style>
    * { box-sizing: border-box; font-family: system-ui, -apple-system, sans-serif; }
    body { margin: 0; background: #f8fafc; color: #1e293b; }
    header { background: #e11d48; color: white; padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center; }
    .filters { max-width: 1150px; margin: 1.5rem auto 0; padding: 0 1rem; display: flex; gap: 0.6rem; flex-wrap: wrap; }
    .filter-btn { background: white; color: #334155; border: 1px solid #cbd5e1; padding: 0.45rem 1rem; border-radius: 999px; cursor: pointer; font-weight: 600; width: auto; }
    .filter-btn.active { background: #e11d48; color: white; border-color: #e11d48; }
    .container { display: grid; grid-template-columns: 2fr 1fr; gap: 2rem; max-width: 1150px; margin: 1.5rem auto; padding: 0 1rem; }
    .menu-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(230px, 1fr)); gap: 1rem; }
    .card, .cart-box { background: white; padding: 1.2rem; border-radius: 12px; box-shadow: 0 2px 10px rgba(0,0,0,0.06); }
    .tag { font-size: 0.72rem; font-weight: 700; padding: 0.15rem 0.45rem; border-radius: 4px; display: inline-block; margin-bottom: 0.4rem; }
    .veg { background: #dcfce7; color: #15803d; border: 1px solid #86efac; }
    .nonveg { background: #fee2e2; color: #b91c1c; border: 1px solid #fca5a5; }
    .card h3 { margin: 0.2rem 0 0.4rem; font-size: 1.05rem; }
    .price { font-weight: 700; color: #0f172a; margin-bottom: 0.9rem; }
    button { background: #e11d48; color: white; border: none; padding: 0.65rem 1rem; border-radius: 8px; cursor: pointer; font-weight: 600; width: 100%; }
    button:hover { background: #be123c; }
    .cart-item { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #f1f5f9; padding: 0.6rem 0; }
    .qty-btn { width: auto; padding: 0.2rem 0.6rem; margin: 0 0.15rem; background: #e2e8f0; color: #1e293b; }
    select, input { width: 100%; padding: 0.55rem; border: 1px solid #cbd5e1; border-radius: 6px; margin-top: 0.5rem; }
    .promo-row { display: flex; gap: 0.5rem; margin: 0.8rem 0; }
    .promo-row input { margin-top: 0; }
    .status-box { margin-top: 1rem; padding: 0.9rem; background: #f0fdf4; border-left: 4px solid #16a34a; border-radius: 6px; display: none; font-size: 0.92rem; line-height: 1.5; }
    @media (max-width: 780px) { .container { grid-template-columns: 1fr; } }
  </style>
</head>
<body>

<header>
  <h2>🍕 CraveCart Food Delivery</h2>
  <span id="cart-badge">Cart: 0 items</span>
</header>

<!-- Dietary & Category Filters -->
<div class="filters">
  <button class="filter-btn active" onclick="filterMenu('All', this)">All Dishes</button>
  <button class="filter-btn" onclick="filterMenu('Veg', this)">🟢 Pure Veg</button>
  <button class="filter-btn" onclick="filterMenu('Non-Veg', this)">🔴 Non-Veg</button>
  <button class="filter-btn" onclick="filterMenu('South Indian', this)">South Indian</button>
  <button class="filter-btn" onclick="filterMenu('Fast Food', this)">Fast Food</button>
</div>

<div class="container">
  <!-- Menu Display -->
  <div>
    <div class="menu-grid" id="menu-grid"></div>
  </div>

  <!-- Cart & Checkout -->
  <div class="cart-box">
    <h3 style="margin-top:0;">🛒 Order Summary</h3>
    <div id="cart-list"><p style="color:#64748b;">No dishes added yet.</p></div>

    <div class="promo-row">
      <input type="text" id="coupon" placeholder="Code: WELCOME20">
      <button style="width:auto;" onclick="applyCoupon()">Apply</button>
    </div>

    <label style="font-size:0.85rem; font-weight:600;">Payment Method</label>
    <select id="payment-method">
      <option value="UPI (GPay / PhonePe)">UPI (GPay / PhonePe)</option>
      <option value="Credit / Debit Card">Credit / Debit Card</option>
      <option value="Cash on Delivery">Cash on Delivery</option>
    </select>

    <h4 id="final-total">Total: ₹0</h4>
    <button onclick="checkout()">Confirm & Pay</button>

    <div class="status-box" id="tracking-box"></div>
  </div>
</div>

<script>
  const dishes = [
    { id: 1, name: "Benne Masala Dosa", price: 110, type: "Veg", category: "South Indian" },
    { id: 2, name: "Chicken Dum Biryani", price: 260, type: "Non-Veg", category: "Main Course" },
    { id: 3, name: "Paneer Tikka Pizza", price: 320, type: "Veg", category: "Fast Food" },
    { id: 4, name: "Crispy Chicken Burger", price: 190, type: "Non-Veg", category: "Fast Food" },
    { id: 5, name: "Filter Coffee & Vada", price: 85, type: "Veg", category: "South Indian" },
    { id: 6, name: "Butter Chicken Bowl", price: 280, type: "Non-Veg", category: "Main Course" }
  ];

  let cart = [];
  let discountRate = 0;

  function renderMenu(items) {
    const grid = document.getElementById("menu-grid");
    grid.innerHTML = items.map(item => `
      <div class="card">
        <span class="tag ${item.type === 'Veg' ? 'veg' : 'nonveg'}">${item.type}</span>
        <small style="color:#64748b; display:block;">${item.category}</small>
        <h3>${item.name}</h3>
        <div class="price">₹${item.price}</div>
        <button onclick="addItem(${item.id})">+ Add Dish</button>
      </div>
    `).join("");
  }

  function filterMenu(criteria, btn) {
    document.querySelectorAll(".filter-btn").forEach(b => b.classList.remove("active"));
    btn.classList.add("active");

    if (criteria === "All") return renderMenu(dishes);
    const filtered = dishes.filter(d => d.type === criteria || d.category === criteria);
    renderMenu(filtered);
  }

  function addItem(id) {
    const found = cart.find(c => c.id === id);
    if (found) found.qty++;
    else cart.push({ ...dishes.find(d => d.id === id), qty: 1 });
    refreshCart();
  }

  function updateQty(id, change) {
    const item = cart.find(c => c.id === id);
    if (!item) return;
    item.qty += change;
    if (item.qty <= 0) cart = cart.filter(c => c.id !== id);
    refreshCart();
  }

  function applyCoupon() {
    const code = document.getElementById("coupon").value.trim().toUpperCase();
    if (code === "WELCOME20") {
      discountRate = 0.20;
      alert("20% discount applied!");
    } else {
      discountRate = 0;
      alert("Invalid coupon. Try WELCOME20.");
    }
    refreshCart();
  }

  function refreshCart() {
    const list = document.getElementById("cart-list");
    const badge = document.getElementById("cart-badge");
    const totalEl = document.getElementById("final-total");

    if (cart.length === 0) {
      list.innerHTML = `<p style="color:#64748b;">No dishes added yet.</p>`;
      badge.innerText = "Cart: 0 items";
      totalEl.innerText = "Total: ₹0";
      return;
    }

    let sum = 0, count = 0;
    list.innerHTML = cart.map(item => {
      sum += item.price * item.qty;
      count += item.qty;
      return `
        <div class="cart-item">
          <div><strong>${item.name}</strong><br><small>₹${item.price} × ${item.qty}</small></div>
          <div>
            <button class="qty-btn" onclick="updateQty(${item.id}, -1)">-</button>
            <button class="qty-btn" onclick="updateQty(${item.id}, 1)">+</button>
          </div>
        </div>
      `;
    }).join("");

    const payable = Math.round(sum * (1 - discountRate));
    badge.innerText = `Cart: ${count} item(s)`;
    totalEl.innerText = discountRate > 0 ? `Total: ₹${payable} (20% Off)` : `Total: ₹${payable}`;
  }

  function checkout() {
    if (cart.length === 0) return alert("Add at least one dish to place an order!");
    const payment = document.getElementById("payment-method").value;
    const box = document.getElementById("tracking-box");

    box.style.display = "block";
    box.innerHTML = `<strong>✅ Order Confirmed!</strong><br>Paid via: ${payment}<br>Status: Sent to Kitchen...`;

    setTimeout(() => { box.innerHTML = `<strong>👨‍🍳 Preparing Your Food</strong><br>Chef is cooking your meal...`; }, 3000);
    setTimeout(() => { box.innerHTML = `<strong>🛵 Out for Delivery</strong><br>Delivery partner is on the way!`; }, 6000);
    setTimeout(() => { box.innerHTML = `<strong>🎉 Delivered!</strong><br>Enjoy your meal!`; }, 9000);

    cart = [];
    discountRate = 0;
    refreshCart();
  }

  renderMenu(dishes);
</script>

</body>
</html>
