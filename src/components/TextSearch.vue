<template>
  <div class="text-search-container">
    <p class="box-title">
      Text-based search
    </p>
    <input
      :value="keyword"
      class="text-search-input"
      type="text"
      @input="$emit('update', $event.target.value)"
      id="textsearch"
      name="textsearch"
      placeholder="Enter keyword ..."
    />

    <button @click.prevent="$emit('search')">
      Search
    </button>
  </div>
</template>

<script>
export default {
  props: ["keyword"],
};
</script>

<style lang="scss" scoped>
.text-search-container {
  display: flex;
  flex-direction: column;

  .box-title {
    font-weight: 600;
  }

  .text-search-input {
    margin-top: 0.25rem;
  }

  button {
    margin-top: 0.25rem;
  }
}
</style>
