# 226-Market-
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MaBoutique</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: 'Segoe UI', sans-serif; background: #f4f5f7; color: #222; }
  header { background: #1a1a2e; color: #fff; padding: 16px 32px; display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; z-index: 10; }
  header h1 { font-size: 1.4rem; }
  .cart-btn { background: #e94560; color: #fff; border: none; padding: 10px 18px; border-radius: 8px; cursor: pointer; font-size: 1rem; }
  .cart-count { background: #fff; color: #e94560; border-radius: 50%; padding: 2px 7px; font-weight: bold; margin-left: 6px; }
  .filters { display: flex; gap: 8px; padding: 20px 32px; flex-wrap: wrap; }
  .filters button { padding: 8px 16px; border: 1px solid #ccc; border-radius: 20px; background: #fff; cursor: pointer; }
  .filters button.active { background: #1a1a2e; color: #fff; }
  .products { display: grid; grid-template-columns: repeat(auto-fill, minmax(230px, 1fr)); gap: 20px; padding: 10px 32px 40px; }
  .card { background: #fff; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,.08); transition: transform .2s; }
  .card:hover { transform: translateY(-4px); }
  .card img { width: 100%; height: 150px; object-fit: cover; }
  .card-body { padding: 14px; }
  .card h3 { font-size: 1rem; margin-bottom: 6px; }
  .price { color: #e94560; font-weight: bold; font-size: 1.1rem; }
  .add-btn { width: 100%; margin-top: 10px; padding: 10px; background: #1a1a2e; color: #fff; border: none; border-radius: 8px; cursor: pointer; }
  .add-btn:hover { background: #e94560; }
  .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,.5); z-index: 20; justify-content: flex-end; }
  .modal.open { display: flex; }
  .cart-panel { background: #fff; width: 100%; max-width: 400px; height: 100%; padding: 24px; display: flex; flex-direction: column; }
  .cart-item { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #eee; }
  .qty button { width: 26px; height: 26px; border: none; border-radius: 4px; cursor: pointer; background: #eee; }
  .cart-total { margin-top: auto; font-size: 1.2rem; font-weight: bold; padding: 16px 0; }
  .checkout-btn { background: #e94560; color: #fff; border: none; padding: 14px; border-radius: 8px; font-size: 1.05rem; cursor: pointer; width: 100%; }
  .close { align-self: flex-end; background: none; border: none; font-size: 1.6rem; cursor: pointer; }
  .toast { position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%); background: #1a1a2e; color: #fff; padding: 12px 24px; border-radius: 8px; opacity: 0; transition: opacity .3s; pointer-events: none; }
  .toast.show { opacity: 1; }
  .empty { text-align: center; color: #888; padding: 40px 0; }
</style>
</head>
<body>

<header>
  <h1>🛍️ MaBoutique</h1>
  <button class="cart-btn" onclick="toggleCart()">Panier<span class="cart-count" id="cartCount">0</span></button>
</header>

<div class="filters" id="filters"></div>
<div class="products" id="products"></div>

<div class="modal" id="cartModal">
  <div class="cart-panel">
    <button class="close" onclick="toggleCart()">×</button>
    <h2>Votre panier</h2>
    <div id="cartItems"></div>
    <div class="cart-total">Total : <span id="cartTotal">0 F</span></div>
    <button class="checkout-btn" onclick="checkout()">Passer la commande</button>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
const products = [
  { id: 1, name: "Smartphone X200", price: 150000, category: "Électronique", img: "https://picsum.photos/seed/phone/400/300" },
  { id: 2, name: "Casque Bluetooth", price: 25000, category: "Électronique", img: "https://picsum.photos/seed/headset/400/300" },
  { id: 3, name: "Ordinateur portable Pro", price: 450000, category: "Informatique", img: "https://picsum.photos/seed/laptop/400/300" },
  { id: 4, name: "Souris sans fil", price: 8000, category: "Informatique", img: "https://picsum.photos/seed/mouse/400/300" },
  { id: 5, name: "T-shirt coton bio", price: 5000, category: "Mode", img: "https://picsum.photos/seed/tshirt/400/300" },
  { id: 6, name: "Sneakers urbaines", price: 32000, category: "Mode", img: "https://picsum.photos/seed/shoes/400/300" },
];

let cart = JSON.parse(localStorage.getItem("cart") || "[]");
let currentCategory = "Tous";

const fmt = n => n.toLocaleString("fr-FR") + " F";

function renderFilters() {
  const cats = ["Tous", ...new Set(products.map(p => p.category))];
  document.getElementById("filters").innerHTML = cats.map(c =>
    `<button class="${c === currentCategory ? "active" : ""}" onclick="filterBy('${c}')">${c}</button>`
  ).join("");
}

function renderProducts() {
  const list = products.filter(p => currentCategory === "Tous" || p.category === currentCategory);
  document.getElementById("products").innerHTML = list.map(p => `
    <div class="card">
      <img src="${p.img}" alt="${p.name}">
      <div class="card-body">
        <h3>${p.name}</h3>
        <p class="price">${fmt(p.price)}</p>
        <button class="add-btn" onclick="addToCart(${p.id})">Ajouter au panier</button>
      </div>
    </div>`).join("");
}

function filterBy(cat) { currentCategory = cat; renderFilters(); renderProducts(); }

function addToCart(id) {
  const item = cart.find(i => i.id === id);
  item ? item.qty++ : cart.push({ ...products.find(p => p.id === id), qty: 1 });
  saveCart(); showToast("Produit ajouté au panier ✓");
}

function changeQty(id, delta) {
  const item = cart.find(i => i.id === id);
  if (!item) return;
  item.qty += delta;
  if (item.qty <= 0) cart = cart.filter(i => i.id !== id);
  saveCart();
}

function saveCart() {
  localStorage.setItem("cart", JSON.stringify(cart));
  renderCart();
}

function renderCart() {
  const count = cart.reduce((s, i) => s + i.qty, 0);
  document.getElementById("cartCount").textContent = count;
  const container = document.getElementById("cartItems");
  container.innerHTML = cart.length === 0
    ? '<p class="empty">Votre panier est vide 🛒</p>'
    : cart.map(i => `
      <div class="cart-item">
        <span>${i.name}<br><small>${fmt(i.price)} × ${i.qty}</small></span>
        <span class="qty">
          <button onclick="changeQty(${i.id},-1)">−</button>
          <button onclick="changeQty(${i.id},1)">+</button>
        </span>
        <strong>${fmt(i.price * i.qty)}</strong>
      </div>`).join("");
  document.getElementById("cartTotal").textContent =
    fmt(cart.reduce((s, i) => s + i.price * i.qty, 0));
}

function toggleCart() {
  document.getElementById("cartModal").classList.toggle("open");
}

function checkout() {
  if (cart.length === 0) return showToast("Votre panier est vide !");
  const total = cart.reduce((s, i) => s + i.price * i.qty, 0);
  alert(`Merci pour votre commande !\nTotal à payer : ${fmt(total)}\n(Un e-mail de confirmation vous sera envoyé.)`);
  cart = [];
  saveCart();
  toggleCart();
}

function showToast(msg) {
  const t = document.getElementById("toast");
  t.textContent = msg;
  t.classList.add("show");
  setTimeout(() => t.classList.remove("show"), 2000);
}

renderFilters();
renderProducts();
renderCart();
</script>
</body>
</html>