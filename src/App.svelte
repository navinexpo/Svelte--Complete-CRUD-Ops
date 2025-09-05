<script>
  import { onMount } from "svelte";

  let products = [];
  let filteredProducts = [];
  let categories = [];
  let selectedCategories = new Set();
  let search = "";
  let sortBy = "";
  let loading = false;

  let formRef;
  // Pagination
  let currentPage = 1;
  const perPage = 8;

  // CRUD form
  let formMode = "add"; // "add" or "update"
  let formProduct = {
    id: null,
    title: "",
    description: "",
    price: "",
    stock: "",
    category: "",
    thumbnail: "",
    rating: "",
  };

  // Fetch categories
  async function fetchCategories() {
    const res = await fetch("https://dummyjson.com/products/category-list");
    categories = await res.json();
  }

  // Fetch products
  async function fetchProducts() {
    loading = true;
    const res = await fetch("https://dummyjson.com/products?limit=100");
    const data = await res.json();
    products = data.products;
    loading = false;
  }

  // Filtered + Sorted products
  $: filteredProducts = products
    .filter((p) => {
      const categoryMatch =
        selectedCategories.size === 0 || selectedCategories.has(p.category);
      const searchMatch = p.title.toLowerCase().includes(search.toLowerCase());
      return categoryMatch && searchMatch;
    })
    .sort((a, b) => {
      if (sortBy === "price-asc") return a.price - b.price;
      if (sortBy === "price-desc") return b.price - a.price;
      return 0;
    });

$: totalPages = Math.ceil(filteredProducts.length / perPage);

// Ensure currentPage is valid
$: if (currentPage > totalPages) currentPage = 1;

$: paginatedProducts = filteredProducts.slice(
  (currentPage - 1) * perPage,
  currentPage * perPage
);
  // Category selection
  function toggleCategory(category) {
    if (selectedCategories.has(category)) selectedCategories.delete(category);
    else selectedCategories.add(category);
    selectedCategories = new Set(selectedCategories);
    currentPage = 1;
  }

  function goToPage(page) {
    currentPage = page;
  }

  // Delete product
  async function deleteProduct(id) {
    loading = true;
    await fetch(`https://dummyjson.com/products/${id}`, { method: "DELETE" });
    products = products.filter((p) => p.id !== id);
    loading = false;
  }

  // Open form for update
  function editProduct(product) {
    console.log("edited", product);
    formMode = "update";
    formProduct = { ...product };
    // Ensure DOM has updated before scrolling
    setTimeout(() => {
      formRef.scrollIntoView({ behavior: "smooth", block: "start" });
    }, 50);
  }

  // Add/Update product
  async function submitForm() {
    loading = true;
    if (formMode === "add") {
      const res = await fetch("https://dummyjson.com/products/add", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(
          formProduct.thumbnail === ""
            ? {
                ...formProduct,
                thumbnail:
                  "https://static.vecteezy.com/system/resources/thumbnails/024/983/914/small_2x/simple-user-default-icon-free-png.png",
              }
            : formProduct
        ),
      });
      const data = await res.json();
      products = [data, ...products];
    } else {
      products = products.map((p) =>
        p.id === formProduct.id ? formProduct : p
      );
    }
    formMode = "add";
    formProduct = {
      id: null,
      title: "",
      description: "",
      price: "",
      stock: "",
      category: "",
      thumbnail: "",
      rating: "",
    };
    loading = false;
    window.scrollTo({ top: 0, behavior: "smooth" }); // scroll to top
  }

  onMount(async () => {
    await fetchCategories();
    await fetchProducts();
  });
</script>

<div class="flex min-h-screen">
  <!-- Sidebar -->
  <div class="sidebar p-6 border-r min-w-[200px] bg-white shadow-lg rounded-lg">
    <h2 class="font-bold text-xl mb-2 text-gray-800">Categories</h2>
    {#each categories as category}
      <div class="mb-1">
        <label
          class="flex items-center cursor-pointer p-2 rounded-md transition-all hover:bg-blue-100 hover:text-blue-700"
          class:bg-blue-100={selectedCategories.has(category)}
          class:text-blue-700={selectedCategories.has(category)}
        >
          <input
            type="checkbox"
            class="form-checkbox h-5 w-5 text-blue-500 rounded border-gray-300 transition-all hover:scale-110 focus:outline-none focus:ring-0"
            on:change={() => toggleCategory(category)}
            checked={selectedCategories.has(category)}
          />
          <span class="ml-3 font-medium">{category}</span>
        </label>
      </div>
    {/each}
  </div>

  <!-- Main Content -->
  <div class="flex-1 p-4 flex flex-col">
    <!-- Search + Sort -->
    <div class="mb-4 flex gap-4 flex-wrap items-center">
      <!-- Search Input -->
      <div class="flex-1 relative">
        <input
          type="text"
          placeholder="Search..."
          bind:value={search}
          class="w-full border border-gray-200 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400 hover:bg-white transition-all"
        />
        <span
          class="absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400"
        >
          🔍
        </span>
      </div>

      <!-- Sort Dropdown -->
      <div class="relative w-64">
        <select
          bind:value={sortBy}
          class="w-full border border-gray-200 rounded-lg p-3 shadow-sm bg-white text-gray-700 appearance-none pr-8 hover:border-blue-400 transition-all cursor-pointer"
        >
          <option value="">Sort By</option>
          <option value="price-asc">Price: Low to High</option>
          <option value="price-desc">Price: High to Low</option>
        </select>
        <!-- Custom arrow for dropdown -->
        <div
          class="pointer-events-none absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400"
        >
          ▼
        </div>
      </div>
    </div>

    {#if loading}
      <div
        class="flex-1 flex items-center justify-center text-lg min-h-[500px] text-bold text-2xl"
      >
        Loading...
      </div>
    {:else}
      <!-- Product Grid -->
      <div
        class="flex-1 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-3 xl:grid-cols-4 gap-4"
      >
        {#if paginatedProducts.length === 0}
          <div
            class="col-span-full flex flex-col items-center justify-center h-full border rounded"
          >
            <img
              width="500"
              src="https://cdn.dribbble.com/userupload/28628569/file/original-edbc9b1a905204e54ac50ca36215712a.jpg"
              alt="No products"
              class="mb-2"
            />
          </div>
        {:else}
          {#each paginatedProducts as product}
            <div
              class="border border-gray-200 rounded-xl shadow-md overflow-hidden flex flex-col max-h-[550px] bg-white transition-transform transform hover:scale-105 hover:shadow-xl duration-300"
            >
              <!-- Product Image -->
              <img
                src={product.thumbnail}
                alt={product.title}
                class="w-full h-48 object-cover"
              />

              <!-- Product Content -->
              <div class="p-4 flex flex-col flex-1">
                <h3 class="font-bold text-lg text-gray-800 mb-1 truncate">
                  {product.title}
                </h3>
                <p class="text-sm text-gray-600 mb-2 line-clamp-3">
                  {product.description}
                </p>

                <div class="mt-auto">
                  <p class="font-semibold text-blue-600 text-lg mb-1">
                    ${product.price}
                  </p>
                  <p class="text-sm text-gray-500">
                    Category: <span class="capitalize">{product.category}</span>
                  </p>
                  <p class="text-sm text-gray-500">Stock: {product.stock}</p>
                  <p class="text-sm text-gray-500">
                    Rating: {product.rating} ⭐
                  </p>
                </div>

                <!-- Buttons -->
                <div class="mt-4 flex gap-3">
                  <button
                    class="flex-1 bg-gradient-to-r from-green-400 to-green-600 hover:from-green-500 hover:to-green-700 text-white font-semibold py-2 rounded-lg shadow-md hover:shadow-lg transition-all duration-300"
                    on:click={() => editProduct(product)}
                  >
                    Edit
                  </button>
                  <button
                    class="flex-1 bg-gradient-to-r from-red-400 to-red-600 hover:from-red-500 hover:to-red-700 text-white font-semibold py-2 rounded-lg shadow-md hover:shadow-lg transition-all duration-300"
                    on:click={() => deleteProduct(product.id)}
                  >
                    Delete
                  </button>
                </div>
              </div>
            </div>
          {/each}
        {/if}
      </div>

      <!-- Pagination -->
      {#if totalPages > 1 && filteredProducts.length > perPage}
        <div class="flex justify-center mt-4 gap-2 flex-wrap">
          {#each Array(totalPages) as _, index}
            <button
              class="px-4 py-2 rounded-lg border border-gray-300 text-gray-700
               hover:bg-blue-500 hover:text-white transition-all duration-200
               {currentPage === index + 1
                ? 'bg-blue-500 text-white border-blue-500'
                : ''}"
              on:click={() => goToPage(index + 1)}
            >
              {index + 1}
            </button>
          {/each}
        </div>
      {/if}
    {/if}

    <!-- Form -->
    {#if paginatedProducts.length !== 0}
      <form
        bind:this={formRef}
        on:submit|preventDefault={submitForm}
        class="m-8 p-6 bg-white rounded-xl shadow-lg border border-gray-200"
      >
        <h3 class="text-xl font-bold mb-4 text-gray-800">
          {formMode === "add" ? "Add Product" : "Update Product"}
        </h3>

        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <input
            type="text"
            placeholder="Title"
            bind:value={formProduct.title}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
            required
          />
          <input
            type="text"
            placeholder="Price"
            bind:value={formProduct.price}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
            required
          />
          <input
            type="text"
            placeholder="Category"
            bind:value={formProduct.category}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
            required
          />
          <input
            type="text"
            placeholder="Thumbnail URL"
            bind:value={formProduct.thumbnail}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
          />
          <input
            type="text"
            placeholder="Stock"
            bind:value={formProduct.stock}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
            required
          />
          <input
            type="text"
            placeholder="Rating"
            bind:value={formProduct.rating}
            class="border border-gray-300 rounded-lg p-3 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
               focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
            required
          />
        </div>

        <textarea
          placeholder="Description"
          bind:value={formProduct.description}
          class="border border-gray-300 rounded-lg p-3 w-full mt-4 shadow-sm bg-gray-50 text-gray-700 placeholder-gray-400
             focus:bg-white focus:border-blue-400 focus:ring-1 focus:ring-blue-400 transition-all"
          required
        ></textarea>

        <button
          type="submit"
          class="mt-4 w-full sm:w-auto bg-blue-500 hover:bg-blue-600 text-white font-semibold px-6 py-3 rounded-lg shadow-md
             transition-all duration-200"
        >
          {formMode === "add" ? "Add Product" : "Update Product"}
        </button>
      </form>
    {/if}
  </div>
</div>
