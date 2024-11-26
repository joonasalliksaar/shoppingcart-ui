<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Shopping Cart</h1>

    <!-- Add Product Section -->
    <div class="mb-4">
      <h2>Add Product</h2>
      <div class="form-inline">
        <input
            v-model="newProduct.name"
            class="form-control mb-2 mr-2"
            placeholder="Product Name"
        />
        <input
            v-model.number="newProduct.price"
            type="number"
            class="form-control mb-2 mr-2"
            placeholder="Price"
        />
        <input
            v-model.number="newProduct.quantity"
            type="number"
            class="form-control mb-2 mr-2"
            placeholder="Quantity"
        />
        <button @click="addProduct" class="btn btn-success mb-2">Add</button>
      </div>
    </div>

    <table class="table">
      <thead>
      <tr>
        <th>Product</th>
        <th>Quantity</th>
        <th>Price</th>
        <th>Action</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in cartItems" :key="item">
        <td>{{ item.name }}</td>
        <td>{{ item.quantity }}</td>
        <td>${{ item.price }}</td>

        <td>
          <button @click="removeProduct(item.name)" class="btn btn-danger btn-sm">Remove</button>
        </td>

      </tr>
      </tbody>
    </table>

    <h3 class="mt-4">Total: ${{ cartTotal.toFixed(2) }}</h3>

  </div>

</template>

<script>
import axios from "axios";

export default {
  data: () => ({
    api: "http://localhost:8089/api/cart",
    cartItems: [],
    cartTotal: 0,
    newProduct: {
      name: "", // Product name input
      price: null, // Product price input
      quantity: null, // Product quantity input
    },
  }),
  methods: {
    fetchCart() {
      axios.all([
        axios
            .get(`${this.api}/get-all-products`)
            .then((res) => (this.cartItems = res.data)),
        axios
            .get(`${this.api}/cart-total-with-tax`)
            .then((res) => (this.cartTotal = res.data)),
      ]);
    },
    removeProduct(name) {
      axios
          .delete(`${this.api}/delete-product-by-name/${encodeURIComponent(name)}`)
          .then(() => {
            alert(`${name} removed successfully!`);
            this.fetchCart(); // Refresh the cart after deletion
          })
          .catch((error) => {
            console.error("Error removing product:", error);
            alert(`Failed to remove ${name}.`);
          });
    },
    addProduct() {
      // Validate the input
      if (!this.newProduct.name || this.newProduct.price <= 0 || this.newProduct.quantity <= 0) {
        alert("Please enter valid product details.");
        return;
      }
      axios
          .post(`${this.api}/add-product`, this.newProduct)
          .then(() => {
            alert(`${this.newProduct.name} added successfully!`);
            this.fetchCart(); // Refresh the cart after adding
            this.newProduct = { name: "", price: null, quantity: null }; // Reset the form
          })
          .catch((error) => {
                console.error("Error adding product:", error);
                alert("Failed to add product.");
          });
    },
  },
  mounted() {
    this.fetchCart();
  },
};
</script>
