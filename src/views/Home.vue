<template>
  <div class="home">
    <h1>{{ message }}</h1>
    <div>
      Search:
      <input type="text" v-model="nameFilter" list="names" />
      <button class="btn btn-primary" v-on:click="sortObject = 'name'">Search By Name</button>
      <datalist id="names">
        <option v-for="product in products">{{ product.title }}</option>
      </datalist>
      <h2>New Product</h2>
      Name:
      <input type="text" v-model="newProductName" />
      <br />
      Description:
      <input type="text" v-model="newProductDescription" />
      <br />
      <small v-if="newProductDescription" v-bind:class="{ 'text-danger': 100 - newProductDescription.length < 5 }">
        {{ 100 - newProductDescription.length }} characters remaining
      </small>
      <br />
      Image_url:
      <input type="text" v-model="newProductImage_url" />
      <br />
      Price:
      <input type="text" v-model="newProductPrice" />
      <br />
      <button v-on:click="CreateProduct()">Create</button>
    </div>

    <div v-for="product in orderBy(filterBy(products, nameFilter, 'name'), sortObject)">
      <h2>Name: {{ product.name }}</h2>
      <img v-bind:src="product.image_url" alt="" />
      <br />
      <button v-on:click="product.showExtraInfo = !product.showExtraInfo">More Info</button>
      <div v-if="product.showExtraInfo">
        <p>Description: {{ product.description }}</p>
        <p>Image_url: {{ product.image_url }}</p>
        <p>Price: {{ product.price }}</p>
        <div>
          <!-- edit -->
          {{ product }}
          <h3>Edit Product</h3>
          Name:
          <input type="text" v-model="product.name" />
          <br />
          Description:
          <input type="text" v-model="product.description" />
          <br />
          Image_url:
          <input type="text" v-model="product.image_url" />
          <br />
          Price:
          <input type="text" v-model="product.price" />
          <br />
          <button v-on:click="updateProduct(product)">Update</button>
          <br />
          <button v-on:click="destroyProduct(product)">Delete</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
img {
  width: 250px;
}
</style>

<script>
import axios from "axios";
import Vue2Filters from "vue2-filters";
export default {
  mixins: [Vue2Filters.mixin],
  data: function() {
    return {
      message: "Products App",
      products: [],
      newProductName: "",
      newProductDescription: "",
      newProductImage_url: "",
      newProductPrice: null,
      nameFilter: "",
      sortObject: "title"
    };
  },
  //show route
  created: function() {
    axios.get("/api/products").then(response => {
      response.data.forEach(product => {
        product.showExtraInfo = false;
      });
      console.log(response.data);
      this.products = response.data;
    });
  },
  methods: {
    relativeDate: function(date) {
      return moment(date).fromNow();
    },
    createProduct: function() {
      var params = {
        name: this.newProductName,
        description: this.newProductDescription,
        image_url: this.newProductImage_url,
        price: this.newProductPrice
      };
      axios
        .post("/api/products", params)
        .then(response => {
          console.log("Success", response.data);
          this.products.push(response.data);
        })
        .catch(error => {
          console.log(error.response.data.errors);
        });
    },
    updateProduct: function(product) {
      var params = {
        name: product.name,
        description: product.description,
        image_url: product.image_url,
        price: product.price
      };
      axios
        .patch("/api/products/" + product.id)
        .then(response => {
          console.log("Success", response.data);
        })
        .catch(error => {
          console.log(error.response.data.errors);
        });
    },
    destroyProduct: function(product) {
      axios.delete(`/api/products/${product.id}`).then(response => {
        console.log("Success", response.data);
        // find index of product object in products array
        var index = this.response.indexOf(product);
        //remove that index from products array
        this.products.splice(index, 1);
      });
    }
  }
};
</script>
