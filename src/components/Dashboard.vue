<template>
  <v-app>
    <v-navigation-drawer app expand-on-hover rail>
      <v-list class="d-flex flex-column justify-space-between fill-height">
        <div>
          <v-list-item
            prepend-icon="mdi-account-circle"
            class="d-flex justify-center"
          >
            {{ userName }}
          </v-list-item>

          <v-divider></v-divider>

          <v-list-item
            prepend-icon="mdi-table"
            title="Create Table"
            @click="currentView = 'create-table'"
          ></v-list-item>

          <v-list-item
            prepend-icon="mdi-table-row"
            title="List Table"
            @click="
              currentView = 'list-table';
              fetchUserTables();
            "
          ></v-list-item>
        </div>

        <v-list-item
          prepend-icon="mdi-door"
          title="Log out"
          @click="logout"
        ></v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container fluid>
        <v-row>
          <v-col>
            <v-card v-if="currentView === 'create-table'" class="pa-4">
              <v-card-title>Create Table</v-card-title>
              <v-card-text>
                <v-text-field
                  v-model="tableName"
                  clearable
                  label="Table Name"
                  variant="solo-filled"
                  @keyup.enter="showAlert = true"
                ></v-text-field>

                <div v-if="showAlert">
                  <div v-for="(column, index) in columns" :key="index">
                    <v-row>
                      <v-col cols="4">
                        <v-autocomplete
                          label="Data Types"
                          :items="dataTypes"
                          v-model="column.dataType"
                        ></v-autocomplete>
                      </v-col>
                      <v-col cols="4">
                        <v-text-field
                          label="Column Name"
                          v-model="column.name"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="4">
                        <v-text-field
                          v-for="(row, rowIndex) in rows"
                          :key="rowIndex"
                          :label="column.name"
                          v-model="row[column.name]"
                        ></v-text-field>
                      </v-col>
                    </v-row>
                  </div>
                  <v-btn class="mt-4" @click="addColumn">Add Column</v-btn>
                  <v-btn class="mt-4" @click="addRow">Add Row</v-btn>
                </div>

                <v-btn class="mt-4" color="primary" @click="createTable">
                  Create Table
                </v-btn>
              </v-card-text>
            </v-card>

            <v-card v-else-if="currentView === 'list-table'" class="pa-4">
              <v-card-title>List Table</v-card-title>
              <v-card-text>
                <v-combobox
                  @click="fetchUserTables"
                  clearable
                  label="Select Table"
                  :items="tables"
                  item-value="tableName"
                  item-text="tableName"
                  v-model="selectedTable"
                  @update:model-value="fetchTableData"
                  variant="solo-filled"
                ></v-combobox>

                <v-table>
                  <thead>
                    <tr>
                      <th v-for="(column, index) in tableColumns" :key="index">
                        {{ column.columnName }}
                      </th>
                      <th>Actions</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(row, rowIndex) in tableRows" :key="rowIndex">
                      <td
                        v-for="(column, colIndex) in tableColumns"
                        :key="colIndex"
                      >
                        {{ row[column.columnName] }}
                      </td>
                      <td>
                        <v-btn
                          icon
                          @click="deleteRow(rowIndex)"
                          class="red--text"
                        >
                          <v-icon>mdi-delete</v-icon>
                        </v-btn>
                        <v-btn icon @click="openEditRowDialog(rowIndex)">
                          <v-icon>mdi-pencil</v-icon>
                        </v-btn>
                      </td>
                    </tr>
                  </tbody>
                </v-table>

                <v-dialog v-model="showAddRowDialog" max-width="500">
                  <v-card>
                    <v-card-title>Add New Row</v-card-title>
                    <v-card-text>
                      <v-form>
                        <v-row
                          v-for="(column, index) in tableColumns"
                          :key="index"
                        >
                          <v-col cols="4">{{ column.columnName }}</v-col>
                          <v-col cols="8">
                            <v-text-field
                              v-model="newRowData[column.columnName]"
                              label="Enter value"
                              :placeholder="`Enter ${column.columnName}`"
                            ></v-text-field>
                          </v-col>
                        </v-row>
                      </v-form>
                    </v-card-text>
                    <v-card-actions>
                      <v-btn color="primary" @click="saveNewRow">Save</v-btn>
                      <v-btn @click="showAddRowDialog = false">Cancel</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <v-dialog v-model="showEditRowDialog" max-width="500">
                  <v-card>
                    <v-card-title>Edit Row</v-card-title>
                    <v-card-text>
                      <v-form>
                        <v-row
                          v-for="(column, index) in tableColumns"
                          :key="index"
                        >
                          <v-col cols="4">{{ column.columnName }}</v-col>
                          <v-col cols="8">
                            <v-text-field
                              v-model="newRowData[column.columnName]"
                              label="Enter value"
                              :placeholder="`Enter ${column.columnName}`"
                            ></v-text-field>
                          </v-col>
                        </v-row>
                      </v-form>
                    </v-card-text>
                    <v-card-actions>
                      <v-btn color="primary" @click="editRow">Save</v-btn>
                      <v-btn @click="showEditRowDialog = false">Cancel</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </v-card-text>
            </v-card>

            <!-- Bottom Navigation -->
            <div v-if="currentView === 'list-table'">
              <v-bottom-navigation :elevation="5">
                <v-btn value="add-row" @click="openAddRowDialog">
                  <v-icon>mdi-plus</v-icon>
                  <span>Add Row</span>
                </v-btn>
              </v-bottom-navigation>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  import { useRouter } from 'vue-router';

  // State variables
  const currentView = ref('create-table');
  const tableName = ref('');
  const showAlert = ref(false);
  const dataTypes = ref(['varChar', 'int', 'decimal', 'dateTime']);
  const columns = ref([{ dataType: '', name: '' }]);
  const rows = ref([{ data: '' }]);
  const tables = ref([]);
  const selectedTable = ref(null);
  const tableRows = ref([]);
  const tableColumns = ref([]);
  const userId = ref(null);
  const showAddRowDialog = ref(false);
  const newRowData = ref({
    columnName: null,
    data: {},
  });
  const showEditRowDialog = ref(false);
  const userName = ref('');
  const router = useRouter();

  const createTable = () => {
    const table = {
      tableName: tableName.value,
      columnsDto: columns.value.map((column) => ({
        columnName: column.name,
        dataType: column.dataType,
      })),

      rowsDto: rows.value.map((row) => {
        const mappedData = columns.value.map(
          (column) => row[column.name] || ''
        );

        return { data: mappedData };
      }),
    };

    axios
      .post('https://localhost:7107/api/Tables/CreateUserTable', table, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        const userId = response.data.userId;
        localStorage.setItem('userId', userId);
        tables.value.push({ tableName: tableName.value });
      })
      .catch((error) => {
        console.error('Error creating table:', error);
      });
  };

  const addColumn = () => {
    columns.value.push({ name: '', dataType: '' });
  };

  const addRow = () => {
    const newRow = {};
    columns.value.forEach((column) => {
      newRow[column.name] = '';
    });
    rows.value.push(newRow);
  };

  // Get user tables names
  const fetchUserTables = () => {
    userId.value = getUserIdFromToken();

    axios
      .get('https://localhost:7107/api/Tables/GetUserTablesNames', {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        tables.value = response.data;
      })
      .catch((error) => {
        console.error('Error fetching user tables:', error);
      });
  };

  //Get table data
  const fetchTableData = () => {
    axios
      .get('https://localhost:7107/api/Tables/GetUserTables', {
        params: { tableName: selectedTable.value },
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        const userTable = response.data[0];

        const userTableId = userTable.id;
        localStorage.setItem('userTableId', userTableId);

        tableColumns.value = userTable.columns.map((column) => ({
          columnName: column.columnName,
        }));
        tableRows.value = userTable.rows.map((row) => {
          const newRow = { id: row.id };
          row.data.forEach((data, index) => {
            const columnName = tableColumns.value[index].columnName;
            newRow[columnName] = data;
          });

          return newRow;
        });
      })
      .catch((error) => {
        console.error('Error fetching table data:', error);
      });
  };

  const getUserIdFromToken = () => {
    const token = localStorage.getItem('jwtToken');
    if (token) {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return payload.sub;
    }
    return null;
  };

  const openAddRowDialog = () => {
    newRowData.value = {
      userTableId: selectedTable.value,
      ...tableColumns.value.reduce((acc, column) => {
        acc[column.columnName] = '';
        return acc;
      }, {}),
    };

    showAddRowDialog.value = true;
  };

  const saveNewRow = () => {
    const userTableId = localStorage.getItem('userTableId');
    const rowData = {
      userTableId: userTableId,
      data: tableColumns.value.map(
        (column) => newRowData.value[column.columnName]
      ),
    };

    axios
      .post('https://localhost:7107/api/Tables/AddRow', rowData, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        tableRows.value.push({ ...newRowData.value });
        showAddRowDialog.value = false;
      })
      .catch((error) => {
        console.error('Error adding row:', error);
      });
  };

  const deleteRow = (rowIndex) => {
    const rowToDelete = tableRows.value[rowIndex];
    const userTableId = localStorage.getItem('userTableId');

    const rowData = tableColumns.value.map(
      (column) => rowToDelete[column.columnName] || ''
    );

    axios
      .delete('https://localhost:7107/api/Tables/DeleteRow', {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
        data: {
          userTableId: userTableId,
          Data: rowData,
        },
      })
      .then((response) => {
        tableRows.value.splice(rowIndex, 1);
      })
      .catch((error) => {
        console.error('Error deleting row:', error);
      });
  };

  const openEditRowDialog = (rowIndex) => {
    const rowToEdit = tableRows.value[rowIndex];
    localStorage.setItem('index', rowIndex);

    const rowData = tableColumns.value.map(
      (column) => rowToEdit[column.columnName] || ''
    );

    newRowData.value = {
      ...tableColumns.value.reduce((acc, column, index) => {
        acc[column.columnName] = rowData[index] || '';
        return acc;
      }, {}),
    };

    localStorage.setItem('rowId', rowToEdit.id);

    showEditRowDialog.value = true;
  };

  const editRow = () => {
    showEditRowDialog.value = true;

    saveEditedRow();
  };

  const saveEditedRow = () => {
    const userTableId = localStorage.getItem('userTableId');
    const rowData = {
      userTableId: userTableId,
      data: tableColumns.value.map(
        (column) => newRowData.value[column.columnName]
      ),
      id: localStorage.getItem('rowId'),
    };

    axios
      .put('https://localhost:7107/api/Tables/UpdateRow', rowData, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        const rowIndex = localStorage.getItem('index');

        tableRows.value[rowIndex] = {
          ...tableRows.value[rowIndex],
          ...newRowData.value,
        };

        showEditRowDialog.value = false;
      })
      .catch((error) => {
        console.error('Error updating row:', error);
      });
  };
  const getUserName = () => {
    axios
      .get('https://localhost:7107/api/Tables/GetUserName', {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('jwtToken')}`,
          'Content-Type': 'application/json',
        },
      })
      .then((response) => {
        userName.value = response.data;
      })
      .catch((error) => {
        console.error('Error getting user name:', error);
      });
  };
  const logout = () => {
    localStorage.removeItem('jwtToken');
    localStorage.removeItem('userId');
    localStorage.removeItem('userTableId');
    router.push({ name: 'login' });
  };

  onMounted(() => {
    getUserName();
  });
</script>
