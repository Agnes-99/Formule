<template>
  <div class="user-profile-page">
    <div v-if="loading" class="loading">Loading profile...</div>
    <div v-else>
      <div class="profile-address-wrapper">
        <!-- Profile Section -->
        <ProfileHeader
          :user="user"
          @update-profile="updateProfile"
          @delete-profile="deleteProfile"
        />

        <!-- Address Section -->
        <section class="address-section">
          <h2>Your Address</h2>

          <button class="add-address-btn" @click="openAddAddressForm">➕ Add New Address</button>

          <div v-if="loadingAddresses" class="loading">Loading addresses...</div>

          <ul v-else-if="addresses.length" class="address-list">
            <li v-for="addr in addresses" :key="addr.id" class="address-card">
              <div class="address-details">
                <p>{{ addr.street }}</p>
                <p>{{ addr.city }}, {{ addr.province }}</p>
                <p>{{ addr.postalCode }}, {{ addr.country }}</p>
              </div>
              <button @click="openEditAddressForm(addr)">Edit</button>
            </li>
          </ul>

          <p v-else class="empty">No addresses found.</p>

          <AddressForm
            v-if="showAddressForm"
            :initialAddress="selectedAddress"
            @saved="onAddressSaved"
            @deleted="onAddressDeleted"
            @cancel="cancelAddressForm"
          />
        </section>
      </div>

    
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import { useUserStore } from '@/store/userStore';
import { storeToRefs } from 'pinia';

import customerAPI from '@/api/users/CustomerService.js';
import ProfileHeader from '@/components/users/ProfileHeader.vue';
import AddressForm from '@/components/generic/AddressForm.vue';
import addressAPI from '@/api/generic/AddressService.js';

const router = useRouter();

const userStore = useUserStore();
const { user } = storeToRefs(userStore); // reactive

const loading = ref(true);
const addresses = ref([]);
const selectedAddress = ref(null);
const showAddressForm = ref(false);
const loadingAddresses = ref(false);

// 🔁 Load user and their addresses
const fetchUser = async () => {
  if (!user.value || !user.value.id) {
    router.push('/auth'); // redirect to login if no user
    return;
  }

  try {
    loading.value = true;
    const data = await customerAPI.getById(user.value.id);
    user.value = data; // update pinia user with fresh data
    await fetchAddresses(data.id);
  } catch (err) {
    console.error("❌ Error fetching profile:", err);
  } finally {
    loading.value = false;
  }
};

const fetchAddresses = async (customerId) => {
  loadingAddresses.value = true;
  try {
    const result = await addressAPI.findByCustomerId(customerId);
    addresses.value = Array.isArray(result) ? result : [result];
  } catch (err) {
    console.error("❌ Error fetching addresses:", err);
  } finally {
    loadingAddresses.value = false;
  }
};

// Profile Actions
const updateProfile = async (updatedUser) => {
  try {
    updatedUser.id = user.value.id;
    const updated = await customerAPI.update(user.value.id, updatedUser);
    user.value = updated;
    alert("✅ Profile updated successfully!");
  } catch (err) {
    console.error("❌ Error updating profile:", err);
    alert("Failed to update profile");
  }
};

const deleteProfile = async () => {
  const confirmed = confirm("Are you sure you want to delete your profile?");
  if (!confirmed) return;

  try {
    await customerAPI.delete(user.value.id);
    userStore.logout();
    alert("✅ Profile deleted successfully!");
    router.push('/');
  } catch (err) {
    console.error("❌ Error deleting profile:", err);
    alert("Failed to delete profile");
  }
};

// Address Form
const openAddAddressForm = () => {
  selectedAddress.value = {
    street: '',
    city: '',
    province: '',
    postalCode: '',
    country: '',
    customer: user.value
  };
  showAddressForm.value = true;
};

const openEditAddressForm = (address) => {
  selectedAddress.value = { ...address };
  showAddressForm.value = true;
};

const onAddressSaved = async () => {
  showAddressForm.value = false;
  await fetchAddresses(user.value.id);
  alert('✅ Address saved successfully!');
};

const onAddressDeleted = async (deletedId) => {
  showAddressForm.value = false;
  addresses.value = addresses.value.filter(addr => addr.id !== deletedId);
  alert('🗑️ Address deleted successfully!');
};

const cancelAddressForm = () => {
  showAddressForm.value = false;
};

onMounted(() => {
  if (!user.value) userStore.initialize(); // initialize from localStorage if needed
  fetchUser();
});
</script>


<style scoped>
:root {
  --cream: #dad7cd;
  --sage: #a3b18a;
  --olive: #588157;
  --forest: #3a5a40;
  --dark-green: #344e41;
  --danger: #b34343;
  --light-bg: #fefefc;
}

/* General Page Styling */
.user-profile-page {
  margin: 1rem auto;
  padding: 2rem 2rem 4rem;
  background-color: var(--sage);
  border-radius: 16px;
  box-shadow: 0 10px 32px rgba(0, 0, 0, 0.08);
  transform: scale(0.90); 
  font-family: 'Inter', 'Segoe UI', sans-serif;
  color: var(--dark-green);
}

/* Loading Indicator */
.loading {
  text-align: center;
  font-size: 1rem;
  color: var(--forest);
  padding: 2rem 0;
}

/* Address Section */
.address-section {
  margin-top: 3rem;
  background-color: white;
  padding: 2rem;
  border-radius: 14px;
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease-in-out;
}

.address-section h2 {
  font-size: 1.4rem;
  margin-bottom: 1.5rem;
  font-weight: 600;
  color: var(--forest);
  letter-spacing: 0.3px;
}

.add-address-btn {
  background-color: var(--olive);
  color: #fff;
  border: none;
  padding: 0.5rem 1.2rem;
  border-radius: 30px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 1.5rem;
  transition: background-color 0.2s ease;
  box-shadow: 0 4px 12px rgba(88, 129, 87, 0.25);
}

.add-address-btn:hover {
  background-color: var(--forest);
}

/* Address List */
.address-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 1rem;
}

.address-card {
  background-color: var(--light-bg);
  border-left: 5px solid var(--sage);
  padding: 1rem 1.4rem;
  border-radius: 10px;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  transition: box-shadow 0.2s ease;
}

.address-card:hover {
  box-shadow: 0 6px 16px rgba(88, 129, 87, 0.15);
}

.address-details {
  flex-grow: 1;
}

.address-card p {
  margin: 0.2rem 0;
  font-size: 0.9rem;
  color: #2c2c2c;
}

.address-card button {
  background-color: var(--forest);
  color: white;
  padding: 0.4rem 1rem;
  font-size: 0.85rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  margin-top: 0.5rem;
  transition: background-color 0.2s ease;
}

.address-card button:hover {
  background-color: var(--dark-green);
}

.empty {
  font-size: 0.95rem;
  color: #777;
  margin-top: 1rem;
  font-style: italic;
  text-align: center;
}


.profile-address-wrapper {
  display: flex;
  gap: 2rem;
  align-items: flex-start;
  /* Let them wrap on smaller screens */
  flex-wrap: wrap;
}

.profile-address-wrapper > * {
  flex: 1 1 40%;
  min-width: 280px;
}

.address-section {
  flex: 1 1 55%;
  min-width: 320px;
}

@media (max-width: 768px) {
  .profile-address-wrapper {
    flex-direction: column;
  }
  .profile-address-wrapper > * {
    flex-basis: auto;
    width: 100%;
  }
}

</style>
