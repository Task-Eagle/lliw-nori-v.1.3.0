# lliw-nori-v.1.3.0
Added a "Save to History" Button, and also a real logic for saving the results.

With the help of Google AI, we replaced everything from the <script> tag, down to the end of the file with this:
<script>
// 1. DATA MANAGEMENT: Load from localStorage or start empty
let deals = JSON.parse(localStorage.getItem('dealHistory')) || [];

const inputs = document.querySelectorAll("input, select, #buyerPays");
inputs.forEach(i => i.addEventListener("input", calculate));

// Run initial render to show saved deals
renderDeals();

function toggleAdvanced(){
  const adv = document.getElementById("advanced");
  adv.style.display = adv.style.display === "none" ? "block" : "none";
}

function setPlatform(){
  const p = document.getElementById("platform").value;
  if(p==="ebay") document.getElementById("fee").value = 13;
  if(p==="amazon") document.getElementById("fee").value = 15;
  calculate(); // Recalculate after platform change
}

function calculate(){
  const buy = parseFloat(document.getElementById("buy").value) || 0;
  const sell = parseFloat(document.getElementById("sell").value) || 0;
  const feePercent = parseFloat(document.getElementById("fee").value) || 13;
  const shippingInput = parseFloat(document.getElementById("shipping").value) || 5;
  const buyerPays = document.getElementById("buyerPays").checked;
  const targetProfit = parseFloat(document.getElementById("targetProfit").value) || 0;

  const shipping = buyerPays ? 0 : shippingInput;
  const fees = sell * (feePercent/100);
  const profit = sell - buy - fees - shipping;

  const roi = buy > 0 ? (profit/buy)*100 : 0;

  let maxBuy = sell - fees - shipping;
  if(targetProfit > 0){
    maxBuy = sell - fees - shipping - targetProfit;
  }

  let decision="", cls="", insight="", confidence=0;

  if(profit>0 && roi>50){
    decision="🔥 BUY"; cls="green";
    insight="Excellent margin. Strong flip.";
    confidence=90;
  } else if(profit>0 && roi>25){
    decision="✅ GOOD"; cls="green";
    insight="Solid deal worth considering.";
    confidence=65;
  } else if(profit>0){
    decision="⚠️ RISKY"; cls="yellow";
    insight="Low margin. Be cautious.";
    confidence=45;
  } else {
    decision="❌ SKIP"; cls="red";
    insight="Not profitable.";
    confidence=15;
  }

  // Update UI
  document.getElementById("decision").textContent = decision;
  document.getElementById("decision").className = "decision "+cls;
  document.getElementById("profit").textContent = "$"+profit.toFixed(2);
  document.getElementById("roi").textContent = roi.toFixed(1)+"%";
  document.getElementById("max").textContent = "$"+maxBuy.toFixed(2);
  document.getElementById("insight").textContent = insight;

  const fill = document.getElementById("fill");
  fill.style.width = confidence+"%";
  if(confidence>70) fill.style.background="var(--green)";
  else if(confidence>40) fill.style.background="var(--yellow)";
  else fill.style.background="var(--red)";
}

// 2. SAVING DATA
function saveCurrentDeal() {
  const buy = parseFloat(document.getElementById("buy").value) || 0;
  const sell = parseFloat(document.getElementById("sell").value) || 0;
  const profit = parseFloat(document.getElementById("profit").textContent.replace('$', '')) || 0;
  const roi = parseFloat(document.getElementById("roi").textContent) || 0;
  const decision = document.getElementById("decision").textContent;

  if (buy === 0 || sell === 0) return alert("Enter buy and sell prices first!");

  const newDeal = {
    id: Date.now(),
    buy,
    sell,
    profit,
    roi,
    decision
  };

  deals.unshift(newDeal); // Add to beginning of list
  updateStorage();
  renderDeals();
}

function deleteDeal(id) {
  deals = deals.filter(d => d.id !== id);
  updateStorage();
  renderDeals();
}

function updateStorage() {
  localStorage.setItem('dealHistory', JSON.stringify(deals));
}

// 3. DISPLAYING DATA
function renderDeals() {
  const list = document.getElementById("dealList");
  list.innerHTML = "";

  let totalProfit = 0;
  let totalROI = 0;

  deals.forEach(d => {
    totalProfit += d.profit;
    totalROI += d.roi;

    const div = document.createElement("div");
    div.className = "deal-card fade";

    div.innerHTML = `
      <div class="deal-top">
        <div class="deal-profit" style="color:${d.profit > 0 ? 'var(--green)' : 'var(--red)'}">$${d.profit.toFixed(2)}</div>
        <div>${d.decision}</div>
      </div>
      <div class="deal-meta">
        ROI: ${d.roi.toFixed(1)}% | Buy: $${d.buy} → Sell: $${d.sell}
      </div>
      <div class="deal-actions">
        <button onclick="deleteDeal(${d.id})" style="background:#fee2e2; color:#b91c1c;">Delete</button>
      </div>
    `;
    list.appendChild(div);
  });

  document.getElementById("totalDeals").textContent = deals.length;
  document.getElementById("avgROI").textContent = deals.length > 0 ? (totalROI / deals.length).toFixed(1) + "%" : "0%";
  document.getElementById("totalProfit").textContent = "$" + totalProfit.toFixed(2);
}
</script>
















We also included this for the green button right before the Result Card ended:
  <!-- Add this inside the result card, at the bottom -->
  <button onclick="saveCurrentDeal()" style="background:var(--green); margin-top:15px;">💾 Save to History</button>
</div>
