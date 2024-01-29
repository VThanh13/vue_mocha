<template>
  <Teleport to="body">
    <ConfirmDelete
      :show="isShowDelete"
      :username="selectedItem.name"
      @cancel="closeDeleteDialog()"
      @delete="deleteUser()"
    ></ConfirmDelete>
  </Teleport>
  <div class="home">
    <div class="user-table">
      <table>
        <caption></caption>
        <thead>
          <tr>
            <th class="th-select"></th>
            <th class="th-no">No</th>
            <th class="th-name">Name</th>
            <th class="th-email">Email</th>
            <th class="th-phone">Phone</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in listAllUsers" :key="item.id">
            <td class="td-select">
              <input v-model="selectedItem" type="radio" :value="item" :disabled="!viewMode" />
            </td>
            <td class="td-id">{{ item.id }}</td>
            <td class="td-name">{{ item.name }}</td>
            <td class="td-email">{{ item.email }}</td>
            <td class="td-phone">{{ item.phone }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="user-form">
      <div class="action-form">
        <button class="btn-add" :disabled="!viewMode" @click="handleModeForm(EModeForm.CREATE)">Add</button>
        <button class="btn-update" :disabled="!viewMode" @click="handleModeForm(EModeForm.EDIT)">Update</button>
        <button class="btn-delete" :disabled="!viewMode" @click="showDeleteDialog()">Delete</button>
      </div>
      <div class="form">
        <form>
          <label for="name">Name</label>
          <input v-model="selectedItem.name" type="text" :required="!viewMode" />
          <span v-if="nameError">{{ nameError }}</span>
          <br />
          <label for="email">Email</label>
          <input v-model="selectedItem.email" type="text" :required="!viewMode" />
          <span v-if="emailError">{{ emailError }}</span>
          <br />

          <label for="position">Position</label>
          <select v-model="selectedItem.code" name="codes">
            <option v-for="item in positionOptions" :key="item.name" :value="item.name">{{ item.name }}</option>
          </select>
          <br />
          <label for="phone">Phone</label>
          <input v-model="selectedItem.phone" type="text" />
          <br />
          <label for="address">Address</label>
          <input v-model="selectedItem.address" type="text" />
        </form>
      </div>
      <div class="confirm-form">
        <button class="btn-cancel" :disabled="viewMode" @click="handleModeForm(EModeForm.VIEW)">Cancel</button>
        <button class="btn-submit" :disabled="viewMode" @click="submitForm()">Submit</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onBeforeMount, computed } from "vue";
import { UserDto } from "@/core/dto/userDto";
import { ConfirmDelete } from "@/components/component";
import { CodeDto } from "@/core/dto/codeDto";
import ApiService from "@/core/services/api.service";
import { ToastUtils } from "@/core/utils/toastUtils";
import { EStatusCode, EModeForm } from "@/core/constants/appConstants";

// VARIABLES

const baseUrl = "https://65b21a099bfb12f6eafcd9ce.mockapi.io/users";
const positionOptions = ref<CodeDto[]>([]);
const listAllUsers = ref<UserDto[]>([]);
const isShowDelete = ref<boolean>(false);

const nameError = ref<string>("");
const emailError = ref<string>("");
const selectedItem = ref<UserDto>(
  new UserDto({
    id: "",
    name: "",
    email: "",
    code: "",
    address: "",
    phone: "",
  }),
);
const modeForm = ref<EModeForm>(EModeForm.VIEW);

const viewMode = computed(() => modeForm.value === EModeForm.VIEW);

// HANDLE FUNCTIONS

const handleModeForm = (mode: EModeForm) => {
  if (mode === EModeForm.CREATE) {
    selectedItem.value = new UserDto({
      id: "",
      name: "",
      email: "",
      code: "",
      address: "",
      phone: "",
    });
  }
  modeForm.value = mode;
};

const showDeleteDialog = () => {
  isShowDelete.value = true;
};

const closeDeleteDialog = () => {
  isShowDelete.value = false;
  modeForm.value = EModeForm.VIEW;
};

const submitForm = () => {
  validateName();
  validateEmail();
  if (emailError.value === "" && nameError.value === "") {
    if (modeForm.value === EModeForm.CREATE) {
      createNewUser();
    } else if (modeForm.value === EModeForm.EDIT) {
      updateUser();
    }
  } else {
    return;
  }
};

const validateName = () => {
  if (selectedItem.value.name === "") {
    nameError.value = "Name is required";
  } else {
    nameError.value = "";
  }
};

const validateEmail = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  if (selectedItem.value.email === "") {
    emailError.value = "Email is required";
  } else if (emailRegex.test(selectedItem.value.email)) {
    emailError.value = "";
  } else {
    emailError.value = "Email address is not valid";
  }
};

// API FUNCTIONS
const getPositionCode = async () => {
  await ApiService.GET("https://65b21a099bfb12f6eafcd9ce.mockapi.io/code")
    .then((res) => {
      if (res.status === EStatusCode.OK) positionOptions.value = res.data;
    })
    .catch((err) => {
      ToastUtils.error(err?.message);
    });
};

const getAllUsers = async () => {
  await ApiService.GET(baseUrl)
    .then((res) => {
      if (res.status === EStatusCode.OK) listAllUsers.value = res.data;
    })
    .catch((err) => {
      ToastUtils.error(err?.message);
    });
};

const updateUser = async () => {
  await ApiService.PUT(baseUrl + "/" + selectedItem.value.id, selectedItem.value)
    .then((res) => {
      if (res.status === EStatusCode.OK) {
        ToastUtils.success("Update user successfully");
        getAllUsers();
      }
    })
    .catch((err) => {
      ToastUtils.error(err?.message);
    });
};

const createNewUser = async () => {
  await ApiService.POST(baseUrl, selectedItem.value)
    .then((res) => {
      if (res.status === EStatusCode.OK) {
        ToastUtils.success("Create user successfully");
        getAllUsers();
      }
    })
    .catch((err) => {
      ToastUtils.error(err?.message);
    });
};

const deleteUser = async () => {
  await ApiService.DELETE(baseUrl + "/" + selectedItem.value.id)
    .then((res) => {
      if (res.status === EStatusCode.OK) {
        ToastUtils.success("Delete user successfully");
        getAllUsers();
        isShowDelete.value = false;
      }
    })
    .catch((err) => {
      ToastUtils.error(err?.message);
    });
};

// ON MOUNTED
onBeforeMount(() => {
  getPositionCode();
  getAllUsers();
});
</script>

<style scoped lang="scss">
.home {
  display: flex;
}

.user-table {
  width: 60%;

  table {
    border-collapse: collapse;
    width: 100%;
    margin: 0;
    border: 1px solid #ddd;
    caption-side: top;
    caption-side: bottom;

    th,
    td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: left;
    }

    .td-select {
      width: 3%;
      text-align: center;
      align-self: center;
    }

    .td-id {
      width: 3%;
      text-align: center;
    }

    .td-name {
      width: 34%;
    }

    .td-email {
      width: 40%;
    }

    .td-phone {
      width: 20%;
    }
  }
}

.user-form {
  width: 40%;
  display: flex;
  flex-direction: column;

  margin: 0 auto;
  padding: 0 20px;

  .form {
    width: 100%;
    align-items: start;
    margin: 20px auto;
    padding: 20px 20px;
    border: 2px solid gray;

    label {
      display: inline-block;
      margin-bottom: 5px;
      font-size: 20px;
      width: 80px;
    }

    input {
      width: 80%;
      padding: 8px;
      margin-bottom: 10px;
      box-sizing: border-box;
      align-self: flex-end;
    }
    select {
      width: 80%;
      padding: 8px;
      margin-bottom: 10px;
      box-sizing: border-box;
      align-self: flex-end;
    }
  }

  .confirm-form {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .action-form {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .btn-add,
  .btn-update,
  .btn-delete,
  .btn-cancel,
  .btn-submit {
    width: 100px;
    height: 40px;
    font-size: 15px;
    background-color: white;
    margin: 10px 0;
    border: 1px solid gray;
    border-radius: 10px;
  }

  .btn-add {
    &:hover {
      background-color: green;
      color: white;
    }
  }

  .btn-update {
    &:hover {
      background-color: blue;
      color: white;
    }
  }

  .btn-delete {
    &:hover {
      background-color: red;
      color: white;
    }
  }

  .btn-cancel {
    &:hover {
      background-color: red;
      color: white;
    }
  }

  .btn-submit {
    &:hover {
      background-color: green;
      color: white;
    }
  }
}
</style>
