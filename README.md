<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Transaction Form</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      padding: 20px;
    }
    form {
      background: #fff;
      padding: 20px;
      border-radius: 10px;
      max-width: 500px;
      margin: auto;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    form input, form select, form button {
      display: block;
      width: 100%;
      margin-bottom: 15px;
      padding: 10px;
      font-size: 1rem;
    }
    button {
      background-color: #28a745;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background-color: #218838;
    }
  </style>
</head>
<body>
  <form id="transactionForm">
    <h2>Transaction Entry</h2>
    <select id="formSource" required>
      <option value="" disabled selected>Select Service Provider</option>
      <option value="Pay Bingo">Pay Bingo</option>
      <option value="Spice Money">Spice Money</option>
      <option value="PayNearby">PayNearby</option>
      <option value="Roinet">Roinet</option>
      <option value="G&G Pay">G&G Pay</option>
      <option value="Manual">Manual Entry</option>
    </select>
    <input type="text" id="manualSource" placeholder="Enter custom provider name" style="display:none;" />

    <input type="datetime-local" id="dateTime" required />
    <input type="text" id="customerName" placeholder="Customer Name" required />
    <input type="number" id="amount" placeholder="Amount" required />
    <input type="text" id="aadhaar" placeholder="Aadhaar Number (12 digits)" required />
    <input type="text" id="rrn" placeholder="Bank RRN (Optional)" />
    <input type="text" id="signature" placeholder="Signature (Optional)" />
    <button type="submit">Submit</button>
  </form>

  <script>
    // Password Prompt
    const password = prompt("Enter admin password:");
    if (password !== "admin123") {
      alert("Access Denied");
      document.body.innerHTML = "";
      throw new Error("Unauthorized");
    }

    // Show manual provider input if 'Manual' selected
    document.getElementById('formSource').addEventListener('change', function () {
      const manualInput = document.getElementById('manualSource');
      manualInput.style.display = this.value === 'Manual' ? 'block' : 'none';
    });

    // Submit to Google Sheets
    function submitToGoogleSheets(txn) {
      fetch('https://script.google.com/macros/s/AKfycbyMNNKEVCV8o0SIDr34_2kGSUcOifuDROEfXZwgI8ZEp8JuATenrnM9klESmlg1pRHk/exec', {
        method: 'POST',
        body: JSON.stringify(txn),
        headers: {
          'Content-Type': 'application/json'
        }
      })
      .then(res => res.text())
      .then(data => {
        console.log('Submitted to Google Sheets:', data);
        alert("Transaction successfully saved!");
        document.getElementById("transactionForm").reset();
        document.getElementById("manualSource").style.display = "none";
      })
      .catch(err => {
        console.error("Failed to submit:", err);
        alert("Failed to submit transaction.\n" + err.message);
      });
    }

    // Form submission handler
    document.getElementById('transactionForm').addEventListener('submit', function (e) {
      e.preventDefault();

      const selectedProvider = document.getElementById('formSource').value;
      const manualSource = document.getElementById('manualSource').value;
      const provider = selectedProvider === 'Manual' ? manualSource : selectedProvider;

      const txn = {
        formSource: provider,
        dateTime: document.getElementById('dateTime').value,
        customerName: document.getElementById('customerName').value,
        amount: document.getElementById('amount').value,
        aadhaar: document.getElementById('aadhaar').value,
        rrn: document.getElementById('rrn').value || '',
        signature: document.getElementById('signature').value || ''
      };

      if (!/^[0-9]{12}$/.test(txn.aadhaar)) {
        alert("Aadhaar must be exactly 12 digits.");
        return;
      }

      submitToGoogleSheets(txn);
    });
  </script>
</body>
</html>
