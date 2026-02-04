# MICROSERVICE-COMMUNICATION
TERMUX CODEING
UPDATE TERMUX PACKAGES
Copy code
Bash
pkg update && pkg upgrade
Press Y if asked.
STEP 3: INSTALL GO (GOLANG)
Copy code
Bash
pkg install golang
Check installation:
Copy code
Bash
go version
You should see something like:
Copy code
Text
go version go1.xx.x linux/arm64
STEP 4: CREATE PROJECT FOLDER
Copy code
Bash
mkdir go-microservices
cd go-microservices
 STEP 5: CREATE PRODUCT SERVICE FILE
Copy code
Bash
nano product_service.go
📌 Paste THIS code ⬇ (FULL & CORRECT)
Copy code
Go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type Product struct {
	ID    int     `json:"id"`
	Name  string  `json:"name"`
	Price float64 `json:"price"`
}

var products []Product

func addProduct(w http.ResponseWriter, r *http.Request) {
	var product Product
	json.NewDecoder(r.Body).Decode(&product)
	products = append(products, product)

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(product)
}

func getProducts(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(products)
}

func main() {
	http.HandleFunc("/product/add", addProduct)
	http.HandleFunc("/products", getProducts)

	fmt.Println("Product Service running on port 8002")
	http.ListenAndServe(":8002", nil)
}
Save:
Press CTRL + O
Press Enter
Press CTRL + X
 STEP 6: RUN PRODUCT SERVICE
Copy code
Bash
go run product_service.go
Expected Output:
Copy code
Text
Product Service running on port 8002
✅ Keep this terminal OPEN
STEP 7: OPEN NEW TERMUX SESSION
Swipe → New Session
Go to project folder again:
Copy code
Bash
cd go-microservices
STEP 8: CREATE ORDER SERVICE FILE
Copy code
Bash
nano order_service.go
📌 Paste THIS code ⬇
Copy code
Go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

type Order struct {
	OrderID   int `json:"order_id"`
	ProductID int `json:"product_id"`
	UserID    int `json:"user_id"`
}

func placeOrder(w http.ResponseWriter, r *http.Request) {
	var order Order
	json.NewDecoder(r.Body).Decode(&order)

	resp, err := http.Get("http://localhost:8002/products")
	if err != nil {
		http.Error(w, "Product service not available", http.StatusInternalServerError)
		return
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println("Fetched Products:", string(body))

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(order)
}

func main() {
	http.HandleFunc("/order/place", placeOrder)

	fmt.Println("Order Service running on port 8003")
	http.ListenAndServe(":8003", nil)
}
Save:
CTRL + O → Enter → CTRL + X
 STEP 9: RUN ORDER SERVICE
Copy code
Bash
go run order_service.go
Expected Output:
Copy code
Text
Order Service running on port 8003
STEP 10: TEST PRODUCT SERVICE
Open third Termux session:
Copy code
Bash
curl -X POST http://localhost:8002/product/add \
-H "Content-Type: application/json" \
-d '{"id":101,"name":"Laptop","price":55000}'
Output:
Copy code
Json
{"id":101,"name":"Laptop","price":55000}
STEP 11: PLACE ORDER (FINAL TEST 🔥)
Copy code
Bash
curl -X POST http://localhost:8003/order/place \
-H "Content-Type: application/json" \
-d '{"order_id":5001,"product_id":101,"user_id":1}'
Output:
Copy code
Json
{"order_id":5001,"product_id":101,"user_id":1}
🖥️ ORDER SERVICE TERMINAL OUTPUT
In Order Service terminal:
Copy code
Text
Fetched Products: [{"id":101,"name":"Laptop","price":55000}]
✔ MICROSERVICE COMMUNICATION SUCCESS 💥
#OUTPUTS
