// One-Tap Fast M-Pesa Logger
async function logMpesaSale(medicationName, unitPrice) {
    // 1. Instantly prompt // ==========================================
// CENTRALIZED PHARMACY TRUCK ENGINE
// ==========================================
const SUPABASE_URL = "YOUR_SUPABASE_URL";
const SUPABASE_KEY = "YOUR_SUPABASE_ANON_KEY";
const supabase = Supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

// 1. FAST M-PESA TRANSACTION PROCESSOR
async function processMpesaSale(medicationName, unitPrice) {
    // Prompt for the total cash received via M-Pesa
    let inputAmount = prompt(`M-PESA SALE: Enter total Ksh received for ${medicationName}:`);
    
    // Exit if cancelled or empty
    if (inputAmount === null || inputAmount.trim() === "") return;
    
    let totalPaid = parseFloat(inputAmount);
    if (isNaN(totalPaid) || totalPaid <= 0) {
        alert("🚨 Invalid amount entered. Please enter a valid number.");
        return;
    }

    // Auto-calculate units sold based on your price
    let quantitySold = Math.round(totalPaid / unitPrice);
    if (quantitySold <= 0) {
        alert("🚨 Amount entered is too low for a single unit sale.");
        return;
    }

    // Prepare data payload
    const payload = {
        medication: medicationName,
        quantity: -quantitySold, // Deducts from inventory
        transaction_type: 'SALE',
        payment_mode: 'MPESA',
        amount_paid: totalPaid,
        created_at: new Date().toISOString()
    };

    // Push to Cloud (Supabase)
    try {
        const { data, error } = await supabase
            .from('pharmacy_transactions')
            .insert([payload]);

        if (error) throw error;

        alert(`✅ Success! Deducted ${quantitySold} units of ${medicationName}. (Ksh ${totalPaid} logged via M-Pesa)`);
        
        // Update the screen instantly
        appendLocalLog(medicationName, quantitySold, totalPaid, 'MPESA');
        updateScreenTotals();

    } catch (err) {
        console.error("Sync Error:", err.message);
        alert("⚠️ Cloud Sync Failed: " + err.message + "\nSaving locally for now.");
        // Fallback local display even if network dips on the road
        appendLocalLog(medicationName, quantitySold, totalPaid, 'MPESA (Offline)');
    }
}

// 2. LIVE INTERFACE LOGGER (Updates the UI dynamically)
function appendLocalLog(medName, qty, cash, mode) {
    const logContainer = document.getElementById('transaction-log');
    if (!logContainer) return;

    const timestamp = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    const logEntry = document.createElement('div');
    logEntry.style.padding = "8px";
    logEntry.style.borderBottom = "1px solid #eee";
    logEntry.style.fontSize = "14px";
    
    logEntry.innerHTML = `
        <strong>[${timestamp}]</strong> ${medName} x${qty} 
        <span style="color: #2e7d32; font-weight: bold;">- Ksh ${cash}</span> 
        <canvas style="display:inline-block; width:8px; height:8px; background:#4caf50; border-radius:50%; margin-left:5px;"></canvas> [${mode}]
    `;
    
    // Insert new entries at the top of the list
    logContainer.insertBefore(logEntry, logContainer.firstChild);
}

// 3. PLACEHOLDER TO REFRESH RUNNING INVENTORY TOTALS
function updateScreenTotals() {
    console.log("Interface updated with latest transactions.");
    // Add logic here to re-fetch or recalculate current truck stock if needed
} quantity or total amount on screen
    let inputAmount = prompt(`M-PESA SALE: How much Ksh was paid for ${medicationName}?`);
    
    // If user clicks cancel or leaves it blank, stop immediately
    if (inputAmount === null || inputAmount.trim() === "") return;
    
    let totalPaid = parseFloat(inputAmount);
    if (isNaN(totalPaid) || totalPaid <= 0) {
        alert("Invalid amount entered!");
        return;
    }

    // 2. Automatically calculate how many units were bought based on unit price
    let quantitySold = Math.round(totalPaid / unitPrice);
    
    // 3. Instantly push to Supabase Cloud
    const { data, error } = await supabase
        .from('pharmacy_transactions')
        .insert([
            { 
                medication: medicationName, 
                quantity: -quantitySold, // Automatically subtracts from your truck stock
                transaction_type: 'SALE',
                payment_mode: 'MPESA',
                amount_paid: totalPaid,
                created_at: new Date().toISOString()
            }
        ]);

    if (error) {
        alert("Sync Failed: " + error.message);
        return;
    }
    
    alert(`Successfully synced! Deducted ${quantitySold} units of ${medicationName} (Ksh ${totalPaid} via M-Pesa).`);
    updateUserInterface(); // Refresh your tablet dashboard screen
}By# My-pharmacy-truck<!DOCTYPE html>
<html lang="en">
<head>
    <meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<meta name="mobile-web-app-capable" content="yes">
<title>Pharmacy Truck</title>
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Truck Pharmacy OS</title>
    <link rel="manifest" href="data:application/manifest+json,{%22name%22:%22Truck%20Pharmacy%20OS%22,%22short_name%22:%22RxTruck%22,%22start_url%22:%22.%22,%22display%22:%22standalone%22,%22background_color%22:%22%23ffffff%22,%22theme_color%22:%22%232563eb%22}">
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 text-gray-800 font-sans pb-12">

    <header class="bg-blue-600 text-white p-4 shadow-md sticky top-0 z-50">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <h1 class="text-xl font-bold">🚛 Pharmacy Truck Manager</h1>
            <div class="flex space-x-3 text-xs md:text-sm">
                <div id="cash-badge" class="bg-blue-800 px-3 py-1 rounded-full font-semibold">Cash: Ksh 0</div>
                <div id="mpesa-badge" class="bg-green-700 px-3 py-1 rounded-full font-semibold">M-Pesa: Ksh 0</div>
            </div>
        </div>
    </header>

    <main class="max-w-4xl mx-auto p-4 space-y-6">

        <section class="grid grid-cols-3 gap-3 bg-white p-4 rounded-xl shadow-sm">
            <div class="bg-green-50 p-3 rounded-lg text-center">
                <p class="text-xs text-gray-500 font-medium">Today's Sales</p>
                <p id="today-sales" class="text-base font-bold text-green-600">Ksh 0</p>
            </div>
            <div class="bg-red-50 p-3 rounded-lg text-center">
                <p class="text-xs text-gray-500 font-medium">Today's Expenses</p>
                <p id="today-expenses" class="text-base font-bold text-red-600">Ksh 0</p>
            </div>
            <div id="profit-box" class="bg-blue-50 p-3 rounded-lg text-center">
                <p class="text-xs text-gray-500 font-medium">Net Profit</p>
                <p id="today-profit" class="text-base font-bold text-blue-600">Ksh 0</p>
            </div>
        </section>

        <section class="bg-white p-6 rounded-xl shadow-sm">
            <h2 class="text-lg font-bold text-blue-600 mb-4">📦 Manage Stock In / Out</h2>
            <form id="stock-form" class="grid grid-cols-1 md:grid-cols-4 gap-3">
                <input type="text" id="prod-name" placeholder="Medicine Name" class="p-2 border rounded w-full" required>
                <input type="number" id="prod-qty" placeholder="Quantity Change (+/-)" class="p-2 border rounded w-full" required>
                <input type="date" id="prod-expiry" class="p-2 border rounded w-full">
                <button type="submit" class="bg-blue-600 text-white p-2 rounded hover:bg-blue-700 font-semibold">Update Stock</button>
            </form>
        </section>

        <section class="bg-white p-6 rounded-xl shadow-sm">
            <h2 class="text-lg font-bold text-green-600 mb-4">💰 Log Sale / Expense</h2>
            <form id="cash-form" class="grid grid-cols-1 md:grid-cols-4 gap-3">
                <input type="text" id="cash-desc" placeholder="Description (e.g., Amoxil sale, Fuel)" class="p-2 border rounded w-full" required>
                <select id="cash-type" class="p-2 border rounded w-full">
                    <option value="sale">Sale (Money In)</option>
                    <option value="expense">Expense (Money Out)</option>
                </select>
                <select id="payment-method" class="p-2 border rounded w-full">
                    <option value="mpesa">M-Pesa</option>
                    <option value="cash">Hard Cash</option>
                </select>
                <input type="number" id="cash-amount" placeholder="Amount (Ksh)" class="p-2 border rounded w-full" required>
                <button type="submit" class="bg-green-600 text-white p-2 rounded hover:bg-green-700 font-semibold md:col-span-4">Log Transaction</button>
            </form>
        </section>

        <section class="bg-white p-6 rounded-xl shadow-sm">
            <h2 class="text-lg font-bold mb-3">📋 Live Truck Inventory</h2>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="border-b bg-gray-50">
                            <th class="p-2">Item</th>
                            <th class="p-2">In-Stock Qty</th>
                            <th class="p-2">Expiry Date</th>
                            <th class="p-2 text-right">Action</th>
                        </tr>
                    </thead>
                    <tbody id="inventory-table">
                        </tbody>
                </table>
            </div>
        </section>

    </main>

    <script>
        // Data controllers using device storage
        let inventory = JSON.parse(localStorage.getItem('truck_inventory')) || {};
        let cashBalance = Number(localStorage.getItem('truck_cash_v2')) || 0;
        let mpesaBalance = Number(localStorage.getItem('truck_mpesa_v2')) || 0;
        
        // P&L Storage arrays for the current day
        let dailyTransactions = JSON.parse(localStorage.getItem('daily_transactions')) || [];

        function updateUI() {
            // Update balance indicators
            document.getElementById('cash-badge').innerText = `Cash: Ksh ${cashBalance.toLocaleString()}`;
            document.getElementById('mpesa-badge').innerText = `M-Pesa: Ksh ${mpesaBalance.toLocaleString()}`;
            
            localStorage.setItem('truck_cash_v2', cashBalance);
            localStorage.setItem('truck_mpesa_v2', mpesaBalance);
            localStorage.setItem('truck_inventory', JSON.stringify(inventory));
            localStorage.setItem('daily_transactions', JSON.stringify(dailyTransactions));

            // Calculate Today's Profits
            let totalSales = 0;
            let totalExpenses = 0;
            dailyTransactions.forEach(tx => {
                if (tx.type === 'sale') totalSales += tx.amount;
                if (tx.type === 'expense') totalExpenses += tx.amount;
            });
            let netProfit = totalSales - totalExpenses;

            document.getElementById('today-sales').innerText = `Ksh ${totalSales.toLocaleString()}`;
            document.getElementById('today-expenses').innerText = `Ksh ${totalExpenses.toLocaleString()}`;
            document.getElementById('today-profit').innerText = `Ksh ${netProfit.toLocaleString()}`;

            // Adjust layout colors dynamically based on profit performance
            let profitBox = document.getElementById('profit-box');
            if (netProfit < 0) {
                profitBox.className = "bg-red-100 p-3 rounded-lg text-center";
            } else if (netProfit > 0) {
                profitBox.className = "bg-green-100 p-3 rounded-lg text-center";
            } else {
                profitBox.className = "bg-blue-50 p-3 rounded-lg text-center";
            }

            // Render inventory table
            const tableBody = document.getElementById('inventory-table');
            tableBody.innerHTML = '';
            
            for (let item in inventory) {
                if (inventory[item].qty <= 0) continue; 
                
                let expiryColor = "text-gray-600";
                if(inventory[item].expiry) {
                    let expDate = new Date(inventory[item].expiry);
                    let today = new Date();
                    let diffTime = expDate - today;
                    let diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
                    if (diffDays < 30) expiryColor = "text-red-600 font-bold animate-pulse";
                }

                tableBody.innerHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-2 font-medium">${item}</td>
                        <td class="p-2">${inventory[item].qty} units</td>
                        <td class="p-2 ${expiryColor}">${inventory[item].expiry || 'N/A'}</td>
                        <td class="p-2 text-right">
                            <button onclick="deleteItem('${item}')" class="text-red-500 text-sm hover:underline">Remove</button>
                        </td>https://cotvqodonpvuznpkrgte.supabase.co/rest/v1/
                    </tr>https://const SUPABASE_URL = "YOUR_SUPABASE_URL";
const SUPABASE_KEY = "YOUR_SUPABASE_ANON_KEY";
const Supabase = Supabase.createClient(SUPABASE_URL, SUPABASE_KEY);.supabase.co/rest/v1/
                `;https://cotvqodonpvuznpkrgte.supabase.co/rest/v1/
            }https://cotvqodonpvuznpkrgte.supabase.co/rest/v1/
        }https://cotvqodonpvuznpkrgte.supabase.co/rest/v1/
https://cotvqodonpvuznpkrgte.supabase.co/rest/v1/
        // Handle Stock Changes
        document.getElementById('stock-form').addEventListener('submit', (e) => {
            e.preventDefault();
            const name = document.getElementById('prod-name').value.trim();
            const qty = parseInt(document.getElementById('prod-qty').value);
            const expiry = document.getElementById('prod-expiry').value;

            if (!inventory[name]) {
                inventory[name] = { qty: 0, expiry: expiry };
            }
            inventory[name].qty += qty;
            if (expiry) inventory[name].expiry = expiry;

            document.getElementById('stock-form').reset();
            updateUI();
        });

        // Handle Upgraded Transaction Entry
        document.getElementById('cash-form').addEventListener('submit', (e) => {
            e.preventDefault();
            const desc = document.getElementById('cash-desc').value.trim();
            const type = document.getElementById('cash-type').value;
            const method = document.getElementById('payment-method').value;
            const amount = parseFloat(document.getElementById('cash-amount').value);

            // Update specific wallet structures
            if (type === 'sale') {
                if (method === 'cash') cashBalance += amount;
                if (method === 'mpesa') mpesaBalance += amount;
            } else {
                if (method === 'cash') cashBalance -= amount;
                if (method === 'mpesa') mpesaBalance -= amount;
            }

            // Log financial performance arrays
            dailyTransactions.push({ desc, type, amount, date: new Date().toLocaleDateString() });

            document.getElementById('cash-form').reset();
            updateUI();
        });

        function deleteItem(name) {
            delete inventory[name];
            updateUI();
        }

        updateUI();
    </script>
</body>
</html>
