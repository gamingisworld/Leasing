from flask import Flask, render_template, request, redirect, url_for
from models import Transaction, db

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///transactions.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db.init_app(app)

@app.before_first_request
def create_tables():
    db.create_all()

@app.route('/')
def index():
    transactions = Transaction.query.all()
    total_income = sum(t.amount for t in transactions if t.type == 'Income')
    total_expenses = sum(t.amount for t in transactions if t.type == 'Expense')
    balance = total_income - total_expenses
    return render_template('index.html', transactions=transactions, income=total_income, expenses=total_expenses, balance=balance)

@app.route('/add', methods=['GET', 'POST'])
def add_transaction():
    if request.method == 'POST':
        description = request.form['description']
        amount = float(request.form['amount'])
        transaction_type = request.form['type']
        new_transaction = Transaction(description=description, amount=amount, type=transaction_type)
        db.session.add(new_transaction)
        db.session.commit()
        return redirect(url_for('index'))
    return render_template('add_transaction.html')

if __name__ == '__main__':
    app.run(debug=True)
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Transaction(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    description = db.Column(db.String(200), nullable=False)
    amount = db.Column(db.Float, nullable=False)
    type = db.Column(db.String(10), nullable=False)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Money Management App</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
</head>
<body>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
{% extends 'base.html' %}

{% block content %}
<h1>Money Management</h1>
<p>Total Income: ${{ income }}</p>
<p>Total Expenses: ${{ expenses }}</p>
<p>Balance: ${{ balance }}</p>

<h2>Transactions</h2>
<ul>
    {% for transaction in transactions %}
    <li>{{ transaction.description }} - ${{ transaction.amount }} ({{ transaction.type }})</li>
    {% endfor %}
</ul>

<a href="{{ url_for('add_transaction') }}">Add Transaction</a>
{% endblock %}
{% extends 'base.html' %}

{% block content %}
<h1>Add Transaction</h1>
<form method="POST">
    <label for="description">Description:</label><br>
    <input type="text" id="description" name="description" required><br><br>
    
    <label for="amount">Amount:</label><br>
    <input type="number" id="amount" name="amount" step="0.01" required><br><br>

    <label for="type">Type:</label><br>
    <select id="type" name="type">
        <option value="Income">Income</option>
        <option value="Expense">Expense</option>
    </select><br><br>
    
    <input type="submit" value="Add Transaction">
</form>
{% endblock %}
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    color: #333;
    margin: 0;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    background: #fff;
    padding: 20px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1, h2 {
    color: #444;
}

a {
    text-decoration: none;
    color: #3498db;
}
Flask==2.0.1
Flask-SQLAlchemy==2.5.1
