# RESTAURANT POS & BILLING SYSTEM

import datetime

GST_PERCENT = 5
SERVICE_CHARGE_PERCENT = 10

# ---------- MENU DATA ----------
menu = {
    "Drinks": {
        "Tea": {"price": 20, "stock": 50},
        "Coffee": {"price": 30, "stock": 40}
    },
    "Main Course": {
        "Burger": {"price": 120, "stock": 25},
        "Pizza": {"price": 250, "stock": 15}
    },
    "Dessert": {
        "Ice Cream": {"price": 80, "stock": 30}
    }
}

# ---------- SALES DATA ----------
daily_sales = []
item_sales = {}

# ---------- COUPONS ----------
coupons = {
    "SAVE10": 10,
    "SAVE20": 20
}

# ---------- LOGIN ----------
ROLES = {
    "manager": "admin123",
    "cashier": "cash123"
}

def login():
    print("\n--- LOGIN ---")
    role = input("Role (manager/cashier): ").lower().strip()
    password = input("Password: ").strip()

    if role in ROLES and ROLES[role] == password:
        print("Login successful!")
        return role
    else:
        print("Invalid credentials")
        return None

# ---------- VIEW MENU ----------
def view_menu():
    print("\n--- MENU ---")
    for category, items in menu.items():
        print(f"\n{category}")
        for name, data in items.items():
            print(f"{name} - ₹{data['price']} | Stock: {data['stock']}")

# ---------- ADD / UPDATE MENU ----------
def manage_menu():
    view_menu()
    cat = input("\nEnter category: ").strip()
    item = input("Item name: ").strip()

    while True:
        price_input = input("Price: ").strip()
        if price_input.isdigit():
            price = int(price_input)
            break
        print("Invalid input! Enter a valid price.")

    while True:
        stock_input = input("Stock: ").strip()
        if stock_input.isdigit():
            stock = int(stock_input)
            break
        print("Invalid input! Enter a valid stock quantity.")

    menu.setdefault(cat, {})
    menu[cat][item] = {"price": price, "stock": stock}
    print("Menu updated successfully!")

# ---------- ORDER TAKING ----------
def take_order():
    order = {}
    while True:
        view_menu()
        cat = input("\nCategory (0 to finish): ").strip()
        if cat == "0":
            break

        item = input("Item name (0 to finish): ").strip()
        if item == "0":
            break

        if cat not in menu or item not in menu[cat]:
            print("Invalid item")
            continue

        while True:
            qty_input = input("Quantity: ").strip()
            if not qty_input.isdigit():
                print("Invalid quantity. Try again.")
                continue
            qty = int(qty_input)
            if qty > menu[cat][item]["stock"]:
                print("Insufficient stock")
                continue
            break

        order[item] = order.get(item, 0) + qty

    return order

# ---------- BILL CALCULATION ----------
def calculate_bill(order):
    subtotal = 0
    for item, qty in order.items():
        for cat in menu:
            if item in menu[cat]:
                subtotal += menu[cat][item]["price"] * qty

    gst = subtotal * GST_PERCENT / 100
    service = subtotal * SERVICE_CHARGE_PERCENT / 100
    total = subtotal + gst + service

    coupon = input("Enter coupon code (or press Enter to skip): ").strip().upper()
    discount = 0
    if coupon:
        if coupon in coupons:
            discount = total * coupons[coupon] / 100
            total -= discount
            print(f"Coupon applied: {coupons[coupon]}% off")
        else:
            print("Invalid coupon code. No discount applied.")

    return subtotal, gst, service, discount, total

# ---------- INVENTORY UPDATE ----------
def update_inventory(order):
    for item, qty in order.items():
        for cat in menu:
            if item in menu[cat]:
                menu[cat][item]["stock"] -= qty
                item_sales[item] = item_sales.get(item, 0) + qty

# ---------- BILL EXPORT ----------
def export_bill(order, bill):
    filename = f"bill_{datetime.datetime.now().strftime('%H%M%S')}.txt"
    with open(filename, "w", encoding="utf-8") as f:  # UTF-8 encoding added
        f.write("RESTAURANT BILL\n\n")
        for item, qty in order.items():
            f.write(f"{item} x {qty}\n")
        f.write(f"\nSubtotal: ₹{bill[0]}")
        f.write(f"\nGST: ₹{bill[1]}")
        f.write(f"\nService Charge: ₹{bill[2]}")
        f.write(f"\nDiscount: ₹{bill[3]}")
        f.write(f"\nTotal: ₹{bill[4]}")

    print(f"Bill exported as {filename}")

# ---------- SALES SUMMARY ----------
def sales_summary():
    print("\n--- DAILY SALES SUMMARY ---")
    total_revenue = sum(s["total"] for s in daily_sales)
    print("Total Revenue:", total_revenue)

    if item_sales:
        most_sold = max(item_sales, key=item_sales.get)
        print("Most Sold Item:", most_sold)

# ---------- MAIN SYSTEM ----------
def main():
    role = login()
    if not role:
        return

    while True:
        print("\n1. View Menu")
        print("2. Take Order")
        if role == "manager":
            print("3. Manage Menu")
            print("4. Sales Summary")
        print("0. Exit")

        choice = input("Choose: ").strip()

        if choice == "1":
            view_menu()

        elif choice == "2":
            order = take_order()
            if not order:
                print("No items ordered.")
                continue

            bill = calculate_bill(order)
            update_inventory(order)

            daily_sales.append({"order": order, "total": bill[4]})

            print("\nFINAL BILL")
            print("Subtotal: ₹", bill[0])
            print("GST: ₹", bill[1])
            print("Service Charge: ₹", bill[2])
            print("Discount: ₹", bill[3])
            print("TOTAL: ₹", bill[4])

            export_bill(order, bill)

        elif choice == "3" and role == "manager":
            manage_menu()

        elif choice == "4" and role == "manager":
            sales_summary()

        elif choice == "0":
            print("Exiting system. Goodbye!")
            break

        else:
            print("Invalid choice")

if __name__ == "__main__":
    main()
