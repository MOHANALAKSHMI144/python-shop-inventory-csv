# **Case Study: Small Shop Inventory and Sales Management (OOP & Menu-Driven)**  

## **Objective**  
This case study helps new programmers practice **file handling, object-oriented programming (OOP), and user input handling** in Python.  

## **Sample Data**

**Products**
```csv
product_id,product_name,price,quantity
101,Soap,20,50
102,Shampoo,120,30
103,Toothpaste,60,40
104,Rice,500,20
105,Sugar,45,25
```

**Sales**
```csv
sale_id,product_id,product_name,quantity_sold,total_price
1,102,Shampoo,2,240
1,101,Soap,3,60
2,104,Rice,1,500
```

## **Features**  
✅ **Manage inventory** (add, view, and update products).  
✅ **Record sales with multiple products** in one sale.  
✅ **Update inventory when a sale is made**.

✅ **Display inventory and sales reports in a **tabular format**.  
✅ **Menu-driven interface** for easy interaction.  

---

## **Class Structure**
- **`Product`**: Represents a product with ID, name, price, and stock quantity.  
- **`Inventory`**: Manages product storage, addition, and updates.  
- **`Sale`**: Represents a single sale, which can include multiple products.  
- **`SalesManager`**: Handles sales processing and record-keeping.  
- **`ShopSystem`**: Provides a menu-driven interface for users.  

---

## **Features Implemented**
✅ **Object-Oriented Design** (Classes: `Product`, `Inventory`, `Sale`, `SalesManager`, `ShopSystem`).  
✅ **File Handling** (`inventory.csv` and `sales.csv`).  
✅ **Exception Handling** .
✅ **Menu-Driven System** (User interacts via a simple interface).  
✅ **Validation** (Check if products exist before selling).  
✅ **Tabular Display** (`tabulate` for inventory and sales reports).  
✅ **One Sale Includes Multiple Products**.  

---

## **Expected Output**
Example:
```
--- Small Shop Management System ---
1. View Inventory
2. Add Product to Inventory
3. Process a Sale
4. View Sales Report
5. Exit
Enter your choice: 3
Enter Sale ID: 1001
Enter Product ID to sell: 101
Enter quantity for Soap: 3
Enter Product ID to sell (or 'done' to finish): done
Sale 1001 recorded successfully.
```

class Product:
    def __init__(self, product_id, product_name, price, quantity):
        self.product_id = product_id
        self.product_name = product_name
        self.price = float(price)
        self.quantity = int(quantity)

   def __str__(self):
        return f"{self.product_id}, {self.product_name}, {self.price}, {self.quantity}"

class Inventory:
    def __init__(self, filename="inventory.csv"):
        self.filename = filename
        self.products = self.load_inventory()

   def load_inventory(self):
        products = {}
        try:
            with open(self.filename, 'r') as file:
                reader = csv.DictReader(file)
                for row in reader:
                    product = Product(row['product_id'], row['product_name'], row['price'], row['quantity'])
                    products[product.product_id] = product
        except FileNotFoundError:
            print("Inventory file not found. Creating a new one.")
        return products

   def save_inventory(self):
        with open(self.filename, 'w', newline='') as file:
            fieldnames = ['product_id', 'product_name', 'price', 'quantity']
            writer = csv.DictWriter(file, fieldnames=fieldnames)
            writer.writeheader()
            for product in self.products.values():
                writer.writerow({
                    'product_id': product.product_id,
                    'product_name': product.product_name,
                    'price': product.price,
                    'quantity': product.quantity
                })

   def add_product(self, product):
        if product.product_id in self.products:
            print("Product ID already exists.")
            return
        self.products[product.product_id] = product
        self.save_inventory()
        print(f"Product {product.product_name} added successfully.")

  def update_product(self, product_id, quantity):
        if product_id not in self.products:
            print("Product not found.")
            return
        self.products[product_id].quantity = quantity
        self.save_inventory()
        print(f"Product {self.products[product_id].product_name} updated successfully.")

   def get_product(self, product_id):
        return self.products.get(product_id)

  def display_inventory(self):
        data = [[p.product_id, p.product_name, p.price, p.quantity] for p in self.products.values()]
        headers = ["Product ID", "Product Name", "Price", "Quantity"]
        print(tabulate(data, headers, tablefmt="grid"))

class Sale:
    def __init__(self, sale_id, items):
        self.sale_id = sale_id
        self.items = items

   def __str__(self):
        return f"Sale ID: {self.sale_id}, Items: {self.items}"

class SalesManager:
    def __init__(self, inventory, filename="sales.csv"):
        self.inventory = inventory
        self.filename = filename
        self.sales = self.load_sales()

   def load_sales(self):
        sales = []
        try:
            with open(self.filename, 'r') as file:
                reader = csv.DictReader(file)
                for row in reader:
                    sales.append(row)
        except FileNotFoundError:
            print("Sales file not found. Creating a new one.")
        return sales

  def save_sales(self):
        with open(self.filename, 'w', newline='') as file:
            fieldnames = ['sale_id', 'product_id', 'product_name', 'quantity_sold', 'total_price']
            writer = csv.DictWriter(file, fieldnames=fieldnames)
            writer.writeheader()
            writer.writerows(self.sales)

  def process_sale(self, sale_id, items):
        for item in items:
            product_id, quantity_sold = item
            product = self.inventory.get_product(product_id)
            if not product:
                print(f"Product ID {product_id} not found.")
                return False
            if product.quantity < quantity_sold:
                print(f"Insufficient stock for {product.product_name}.")
                return False
        for item in items:
            product_id, quantity_sold = item
            product = self.inventory.get_product(product_id)
            total_price = product.price * quantity_sold
            self.sales.append({
                'sale_id': sale_id,
                'product_id': product_id,
                'product_name': product.product_name,
                'quantity_sold': quantity_sold,
                'total_price': total_price
            })
            self.inventory.update_product(product_id, product.quantity - quantity_sold)
        self.save_sales()
        print(f"Sale {sale_id} recorded successfully.")
        return True

   def display_sales_report(self):
        if not self.sales:
            print("No sales recorded.")
            return
        headers = self.sales[0].keys()
        data = [list(sale.values()) for sale in self.sales]
        print(tabulate(data, headers, tablefmt="grid"))

class ShopSystem:
    def __init__(self):
        self.inventory = Inventory()
        self.sales_manager = SalesManager(self.inventory)

   def run(self):
        while True:
            print("\n--- Small Shop Management System ---")
            print("1. View Inventory")
            print("2. Add Product to Inventory")
            print("3. Process a Sale")
            print("4. View Sales Report")
            print("5. Exit")
            choice = input("Enter your choice: ")

   if choice == '1':
                self.inventory.display_inventory()
            elif choice == '2':
                product_id = input("Enter Product ID: ")
                product_name = input("Enter Product Name: ")
                price = float(input("Enter Price: "))
                quantity = int(input("Enter Quantity: "))
                product = Product(product_id, product_name, price, quantity)
                self.inventory.add_product(product)
            elif choice == '3':
                sale_id = input("Enter Sale ID: ")
                items = []
                while True:
                    product_id = input("Enter Product ID to sell (or 'done' to finish): ")
                    if product_id.lower() == 'done':
                        break
                    quantity_sold = int(input(f"Enter quantity for {self.inventory.get_product(product_id).product_name}: "))
                    items.append((product_id, quantity_sold))
                self.sales_manager.process_sale(sale_id, items)
            elif choice == '4':
                self.sales_manager.display_sales_report()
            elif choice == '5':
                break
            else:
                print("Invalid choice. Please try again.")

if __name__ == "__main__":
    shop_system = ShopSystem()
    shop_system.run()
