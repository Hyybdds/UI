<template>
  <div class="p-6">
    <h1 class="text-2xl font-bold mb-6">User Permissions</h1>

    <!-- Search Input and Add Button -->
    <div class="mb-4 flex justify-between items-center">
      <div class="relative">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search by Module or Privilege..."
          class="border p-2 w-full rounded-lg"
        />
      </div>
      <button @click="showAddModal" class="bg-blue-500 text-white px-4 py-2 rounded">
        Add Permission
      </button>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto">
      <table class="min-w-full bg-white shadow-md rounded-lg overflow-hidden">
        <thead class="bg-gray-200">
          <tr>
            <th class="py-2 px-4">Privilege Name</th>
            <th class="py-2 px-4">ID</th>
            <th class="py-2 px-4">Modules</th>
            <th class="py-2 px-4">Create</th>
            <th class="py-2 px-4">Update</th>
            <th class="py-2 px-4">Cancel</th>
            <th class="py-2 px-4">View</th>
            <th class="py-2 px-4">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(permission, index) in paginatedPermissions" :key="index" class="text-center">
            <td class="border-t px-4 py-2">{{ permission.priv_n }}</td>
            <td class="border-t px-4 py-2">{{ permission.id }}</td>
            <td class="border-t px-4 py-2">{{ permission.module }}</td>
            <td class="border-t px-4 py-2">
              <input type="checkbox" v-model="permission.canCreate" class="form-checkbox" />
            </td>
            <td class="border-t px-4 py-2">
              <input type="checkbox" v-model="permission.canUpdate" class="form-checkbox" />
            </td>
            <td class="border-t px-4 py-2">
              <input type="checkbox" v-model="permission.canCancel" class="form-checkbox" />
            </td>
            <td class="border-t px-4 py-2">
              <input type="checkbox" v-model="permission.canView" class="form-checkbox" />
            </td>
            <td class="border-t px-4 py-2">
              <font-awesome-icon 
                icon="fas fa-eye" 
                class="cursor-pointer" 
                @click="viewPermission(permission)" 
              />
              <font-awesome-icon 
                icon="fas fa-pencil-alt" 
                class="cursor-pointer ml-2" 
                @click="editPermission(permission)" 
              />
              <font-awesome-icon 
                icon="fas fa-trash" 
                class="cursor-pointer text-red-600 ml-2" 
                @click="deletePermission(permission)" 
              />
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="mt-4 flex justify-between">
      <button
        @click="prevPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
      >
        Previous
      </button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button
        @click="nextPage"
        :disabled="currentPage === totalPages"
        class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
      >
        Next
      </button>
    </div>

    <!-- Modal for Adding Permission -->
    <div v-if="showModal" class="fixed inset-0 flex items-center justify-center bg-gray-900 bg-opacity-50">
      <div class="bg-white p-6 rounded-lg shadow-md">
        <h2 class="text-lg font-semibold mb-4">Add New Permission</h2>
        <form @submit.prevent="handleSubmit">
          <div class="mb-4">
            <label for="privilegeName" class="block text-sm font-medium text-gray-700">Privilege Name:</label>
            <input
              v-model="newPermission.priv_n"
              type="text"
              id="privilegeName"
              name="privilegeName"
              class="border p-2 w-full rounded-lg"
              required
            />
          </div>
          <div class="mb-4">
            <label for="moduleId" class="block text-sm font-medium text-gray-700">Module ID:</label>
            <input
              v-model="newPermission.module"
              type="text"
              id="moduleId"
              name="moduleId"
              class="border p-2 w-full rounded-lg"
              required
            />
          </div>
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700">Permissions:</label>
            <div class="flex items-center space-x-4">
              <label>
                <input type="checkbox" v-model="newPermission.canCreate" class="form-checkbox" />
                <span class="ml-2">Create</span>
              </label>
              <label>
                <input type="checkbox" v-model="newPermission.canUpdate" class="form-checkbox" />
                <span class="ml-2">Update</span>
              </label>
              <label>
                <input type="checkbox" v-model="newPermission.canCancel" class="form-checkbox" />
                <span class="ml-2">Cancel</span>
              </label>
              <label>
                <input type="checkbox" v-model="newPermission.canView" class="form-checkbox" />
                <span class="ml-2">View</span>
              </label>
            </div>
          </div>
          <div class="flex justify-end">
            <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">
              Add
            </button>
            <button @click="closeModal" class="ml-2 bg-gray-300 text-gray-700 px-4 py-2 rounded">
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faEye, faPencilAlt, faTrash } from '@fortawesome/free-solid-svg-icons';

// Add the icons to the library
library.add(faEye, faPencilAlt, faTrash);

export default {
  components: {
    FontAwesomeIcon,
  },
  data() {
    return {
      searchQuery: '',
      currentPage: 1,
      itemsPerPage: 5,
      permissions: [
        // Sample existing permissions for demonstration
        { id: 1, priv_n: 'Edit User', module: 'User Management', canCreate: false, canUpdate: true, canCancel: false, canView: true, isActive: true },
        { id: 2, priv_n: 'Delete User', module: 'User Management', canCreate: false, canUpdate: false, canCancel: true, canView: true, isActive: false },
      ],
      showModal: false,
      newPermission: {
        priv_n: '',
        module: '',
        canCreate: false,
        canUpdate: false,
        canCancel: false,
        canView: false,
      },
    };
  },
  computed: {
    filteredPermissions() {
      return this.permissions.filter((permission) => {
        const search = this.searchQuery.toLowerCase();
        return (
          permission.priv_n.toLowerCase().includes(search) ||
          permission.module.toLowerCase().includes(search)
        );
      });
    },
    paginatedPermissions() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.filteredPermissions.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.filteredPermissions.length / this.itemsPerPage);
    },
  },
  methods: {
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    showAddModal() {
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
    handleSubmit() {
      // Add new permission with isActive set to true
      this.permissions.push({ ...this.newPermission, isActive: true, id: this.permissions.length + 1 });
      this.newPermission = {
        priv_n: '',
        module: '',
        canCreate: false,
        canUpdate: false,
        canCancel: false,
        canView: false,
      };
      this.showModal = false;
    },
    viewPermission(permission) {
      // View permission details (for demonstration)
      alert(`Viewing Permission: ${permission.priv_n}`);
    },
    editPermission(permission) {
      // Edit permission functionality (for demonstration)
      alert(`Editing Permission: ${permission.priv_n}`);
    },
    deletePermission(permission) {
      // Delete permission functionality (for demonstration)
      const index = this.permissions.indexOf(permission);
      if (index !== -1) {
        this.permissions.splice(index, 1);
      }
    },
  },
};
</script>

<style scoped>
/* Add any additional styling here if needed */
</style>
