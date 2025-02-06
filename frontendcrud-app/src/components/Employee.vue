<template>
    <div class="min-h-screen bg-gradient-to-br from-violet-50 via-purple-50 to-fuchsia-50 p-6">
        <!-- Animated Header Section -->
        <div class="container mx-auto max-w-7xl">
            <div class="text-center mb-10">
                <h1
                    class="text-4xl font-bold bg-gradient-to-r from-violet-600 to-fuchsia-600 bg-clip-text text-transparent">
                    Team Directory
                </h1>
                <p class="text-gray-600 mt-2">Streamline your workforce management</p>
            </div>

            <!-- Quick Stats -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
                <div class="bg-white/80 backdrop-blur rounded-xl p-4 shadow-lg border border-violet-100">
                    <div class="flex items-center gap-4">
                        <div class="w-12 h-12 bg-violet-100 rounded-full flex items-center justify-center">
                            <v-icon color="violet">mdi-account-group</v-icon>
                        </div>
                        <div>
                            <p class="text-sm text-gray-600">Total Team</p>
                            <p class="text-2xl font-bold text-violet-600">{{ Object.keys(result).length }}</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Employee Form Card -->
            <div class="bg-white/90 backdrop-blur rounded-xl shadow-lg mb-8">
                <div class="p-6 bg-gradient-to-r from-violet-600 to-fuchsia-600 rounded-t-xl">
                    <h2 class="text-xl font-semibold text-white">{{ employee.id ? 'Update Profile' : 'Add Team Member'
                        }}</h2>
                </div>

                <form @submit.prevent="save" class="p-6 grid gap-6">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <!-- Name Field -->
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Name</label>
                            <v-text-field v-model="employee.name" placeholder="Enter full name"
                                class="rounded-lg bg-gray-50" variant="outlined" dense>
                                <template v-slot:prepend>
                                    <v-icon>mdi-account</v-icon>
                                </template>
                            </v-text-field>
                        </div>

                        <!-- Address Field -->
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Address</label>
                            <v-text-field v-model="employee.address" placeholder="Enter address"
                                class="rounded-lg bg-gray-50" variant="outlined" dense>
                                <template v-slot:prepend>
                                    <v-icon>mdi-map-marker</v-icon>
                                </template>
                            </v-text-field>
                        </div>

                        <!-- Phone Field -->
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Phone</label>
                            <v-text-field v-model="employee.phone" placeholder="Enter phone number"
                                class="rounded-lg bg-gray-50" variant="outlined" dense>
                                <template v-slot:prepend>
                                    <v-icon>mdi-phone</v-icon>
                                </template>
                            </v-text-field>
                        </div>
                    </div>

                    <!-- Action Buttons -->
                    <div class="flex gap-4">
                        <v-btn type="submit" color="primary" class="px-6 rounded-full" :loading="loading">
                            {{ employee.id ? 'Update' : 'Save' }}
                        </v-btn>

                        <v-btn v-if="employee.id" @click="resetForm" color="grey" variant="outlined"
                            class="rounded-full">
                            Cancel
                        </v-btn>
                    </div>
                </form>
            </div>

            <!-- Employee Table -->
            <div class="bg-white/90 backdrop-blur rounded-xl shadow-lg overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-4 text-left text-sm font-medium text-gray-600">Profile</th>
                                <th class="px-6 py-4 text-left text-sm font-medium text-gray-600">Contact Info</th>
                                <th class="px-6 py-4 text-right text-sm font-medium text-gray-600">Actions</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-100">
                            <tr v-for="emp in result" :key="emp.id" class="hover:bg-gray-50/50 transition-colors">
                                <td class="px-6 py-4">
                                    <div class="flex items-center gap-4">
                                        <div
                                            class="w-10 h-10 rounded-full bg-gradient-to-br from-violet-500 to-fuchsia-500 flex items-center justify-center text-white font-medium">
                                            {{ emp.name.charAt(0) }}
                                        </div>
                                        <div>
                                            <p class="font-medium text-gray-900">{{ emp.name }}</p>
                                            <p class="text-sm text-gray-500">{{ emp.address }}</p>
                                        </div>
                                    </div>
                                </td>
                                <td class="px-6 py-4">
                                    <div class="flex items-center gap-2">
                                        <v-icon size="small" color="gray">mdi-phone</v-icon>
                                        <span class="text-gray-600">{{ emp.phone }}</span>
                                    </div>
                                </td>
                                <td class="px-6 py-4">
                                    <div class="flex gap-2 justify-end">
                                        <v-btn color="primary" icon small @click="edit(emp)"
                                            class="hover:scale-110 transition-transform">
                                            <v-icon>mdi-pencil</v-icon>
                                        </v-btn>

                                        <v-btn color="error" icon small @click="remove(emp)"
                                            class="hover:scale-110 transition-transform">
                                            <v-icon>mdi-delete</v-icon>
                                        </v-btn>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'redaxios';

export default {
    name: 'Employee',
    data() {
        return {
            result: {},
            employee: {
                id: '',
                name: '',
                address: '',
                phone: ''
            },
            loading: false,
            showDeleteDialog: false,
            employeeToDelete: null
        }
    },

    created() {
        this.EmployeeLoad();
    },

    methods: {
        async EmployeeLoad() {
            try {
                const { data } = await axios.get('http://127.0.0.1:8000/api/employee');
                this.result = data;
            } catch (error) {
                console.error('Error loading employees:', error);
            }
        },

        async save() {
            this.loading = true;
            try {
                if (this.employee.id) {
                    await this.updateData();
                } else {
                    await this.saveData();
                }
            } finally {
                this.loading = false;
            }
        },

        async saveData() {
            try {
                await axios.post("http://127.0.0.1:8000/api/employee", this.employee);
                this.EmployeeLoad();
                this.resetForm();
            } catch (error) {
                console.error('Error saving employee:', error);
            }
        },

        edit(employee) {
            this.employee = { ...employee };
        },

        async updateData() {
            try {
                const url = `http://127.0.0.1:8000/api/employee/${this.employee.id}`;
                await axios.put(url, this.employee);
                this.resetForm();
                this.EmployeeLoad();
            } catch (error) {
                console.error('Error updating employee:', error);
            }
        },

        confirmDelete(employee) {
            this.employeeToDelete = employee;
            this.showDeleteDialog = true;
        },

        async handleDelete() {
            if (!this.employeeToDelete) return;

            try {
                const url = `http://127.0.0.1:8000/api/employee/${this.employeeToDelete.id}`;
                await axios.delete(url);
                this.showDeleteDialog = false;
                this.employeeToDelete = null;
                this.EmployeeLoad();
            } catch (error) {
                console.error('Error deleting employee:', error);
            }
        },

        resetForm() {
            this.employee = {
                id: '',
                name: '',
                address: '',
                phone: ''
            };
        }
    }
}
</script>

<style>
.v-text-field input {
    background-color: white !important;
}

.v-btn {
    text-transform: none !important;
}

.v-data-table {
    background: transparent !important;
}
</style>