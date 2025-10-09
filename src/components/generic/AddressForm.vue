<template>
  <div class="address-form-container">
    <h3 class="form-title">{{ isEditMode ? 'Update Address' : 'Add New Address' }}</h3>

    <form @submit.prevent="handleSubmit" class="form-grid">
      <div class="form-group">
        <label for="street">Street</label>
        <input id="street" v-model="address.street" required />
      </div>

      <div class="form-group">
        <label for="city">City</label>
        <input id="city" v-model="address.city" required />
      </div>

      <div class="form-group">
        <label for="province">Province</label>
        <input id="province" v-model="address.province" required />
      </div>

      <div class="form-group">
        <label for="postalCode">Postal Code</label>
        <input id="postalCode" v-model="address.postalCode" required />
      </div>

      <div class="form-group">
        <label for="country">Country</label>
        <input id="country" v-model="address.country" required />
      </div>

      <div class="form-actions full-width">
        <button type="submit" class="btn primary">
          {{ isEditMode ? 'Update' : 'Add' }}
        </button>

        <button
          v-if="isEditMode"
          type="button"
          class="btn danger"
          @click="handleDelete"
        >
          Delete
        </button>

        <button type="button" class="btn secondary" @click="$emit('cancel')">
          Cancel
        </button>
      </div>
    </form>
  </div>
</template>

<script>
import addressAPI from '@/api/generic/AddressService.js';

export default {
  name: 'AddressForm',

  props: {
    initialAddress: {
      type: Object,
      default: () => ({
        street: '',
        city: '',
        province: '',
        postalCode: '',
        country: '',
        customer: null,
      }),
    },
  },

  data() {
    return {
      address: { ...this.initialAddress },
    };
  },

  computed: {
    isEditMode() {
      return Boolean(this.address.id);
    },
  },

  methods: {
    async handleSubmit() {
      try {
        const result = this.isEditMode
          ? await addressAPI.update(this.address)
          : await addressAPI.create(this.address);
        this.$emit('saved', result);
      } catch (error) {
        console.error('Error saving address:', error);
        alert('Failed to save address.');
      }
    },

    async handleDelete() {
      if (!this.address.id || !confirm('Are you sure you want to delete this address?')) return;

      try {
        await addressAPI.delete(this.address.id);
        this.$emit('deleted', this.address.id);
      } catch (error) {
        console.error('Error deleting address:', error);
        alert('Failed to delete address.');
      }
    },
  },
};
</script>

<style scoped>
:root {
  --cream: #dad7cd;
  --sage: #a3b18a;
  --olive: #588157;
  --forest: #3a5a40;
  --dark-green: #344e41;
}

.address-form-container {
  background-color: var(--sage);
  padding: 1.5rem 2rem;
  border-radius: 10px;
  max-width: 520px;
  margin: 3rem auto;
  box-shadow: 0 6px 24px rgba(52, 78, 65, 0.2);
  font-family: 'Inter', 'Segoe UI', sans-serif;
  transform: scale(0.95); 
}

.form-title {
  text-align: center;
  color: var(--dark-green);
  margin-bottom: 1.2rem;
  font-size: 1.35rem;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem 1.2rem;
}

.form-group.full-width,
.form-actions.full-width {
  grid-column: span 2;
}


.form-group label {
  margin-bottom: 0.25rem;
  font-weight: 500;
  color: var(--dark-green);
  font-size: 0.85rem;
}

.form-group input {
  padding: 0.5rem 0.75rem;
  font-size: 0.9rem;
  border: 1px solid var(--sage);
  border-radius: 5px;
  background-color: white;
  color: #2c2c2c;
  transition: all 0.2s ease;
}

.form-group input:focus {
  border-color: var(--forest);
  box-shadow: 0 0 0 3px rgba(88, 129, 87, 0.15);
  outline: none;
}

.form-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn {
  padding: 0.45rem 1.1rem;
  font-size: 0.9rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.25s ease, box-shadow 0.2s ease;
}

.btn.primary {
  background-color: var(--olive);
  color: white;
}

.btn.primary:hover {
  background-color: var(--forest);
  box-shadow: 0 2px 8px rgba(58, 90, 64, 0.3);
}

.btn.secondary {
  background-color: var(--sage);
  color: var(--dark-green);
}

.btn.secondary:hover {
  background-color: #b7c7a4;
  box-shadow: 0 2px 6px rgba(163, 177, 138, 0.3);
}

.btn.danger {
  background-color: #b34343;
  color: white;
}

.btn.danger:hover {
  background-color: #962e2e;
  box-shadow: 0 2px 6px rgba(179, 67, 67, 0.3);
}

.btn:active {
  transform: scale(0.97);
}
</style>


