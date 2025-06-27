<template>
  <div class="contact-app">
    <div class="search-box">
      <InputSearch v-model="searchText" />
    </div>

    <div class="content-container">
      <!-- Danh sách liên hệ -->
      <div class="contact-list-panel">
        <div class="panel-header">
          <h2 class="panel-title">
            <i class="fas fa-address-book"></i>
            Danh bạ
            <span class="counter-badge">{{ filteredContactsCount }}</span>
          </h2>
        </div>

        <div class="list-wrapper">
          <ContactList
            v-if="filteredContactsCount > 0"
            :contacts="filteredContacts"
            v-model:activeIndex="activeIndex"
          />
          <div v-else class="no-contacts">
            <i class="fas fa-user-slash"></i>
            <p>Danh bạ trống</p>
          </div>
        </div>

        <div class="control-buttons">
          <button class="control-btn refresh" @click="refreshList()">
            <i class="fas fa-sync-alt"></i> Làm mới
          </button>
          <button class="control-btn add" @click="goToAddContact">
            <i class="fas fa-user-plus"></i> Thêm mới
          </button>
          <button class="control-btn delete" @click="removeAllContacts">
            <i class="fas fa-trash-alt"></i> Xóa tất cả
          </button>
        </div>
      </div>

      <!-- Chi tiết liên hệ -->
      <div class="contact-detail-panel" v-if="activeContact">
        <div class="panel-header">
          <h2 class="panel-title">
            <i class="fas fa-id-card"></i>
            Chi tiết Liên hệ
          </h2>
        </div>
        <ContactCard :contact="activeContact" />
        <router-link
          :to="{
            name: 'contact.edit',
            params: { id: activeContact._id },
          }"
        >
          <span class="mt-2 badge badge-warning">
            <i class="fas fa-edit"></i> Hiệu chỉnh</span
          >
        </router-link>
      </div>
    </div>
  </div>
</template>

<style scoped>
.contact-app {
  width: 1200px;
  margin: 0 auto;
  padding: 30px;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.search-box {
  width: 60%;
  margin: 0 auto 30px;
}

.content-container {
  display: flex;
  gap: 30px;
}

.contact-list-panel,
.contact-detail-panel {
  flex: 1;
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.panel-header {
  border-bottom: 1px solid #eaeef2;
  padding-bottom: 15px;
  margin-bottom: 20px;
}

.panel-title {
  font-size: 20px;
  color: #2c3e50;
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 0;
}

.counter-badge {
  background: #42b983;
  color: white;
  border-radius: 50%;
  width: 28px;
  height: 28px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  margin-left: 8px;
}

.list-wrapper {
  height: 500px;
  overflow-y: auto;
  margin-bottom: 25px;
  padding-right: 10px;
}

.no-contacts {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #9aa5b1;
  font-size: 18px;
}

.no-contacts i {
  font-size: 50px;
  margin-bottom: 15px;
  opacity: 0.5;
}

.control-buttons {
  display: flex;
  justify-content: space-between;
  gap: 15px;
}

.control-btn {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  cursor: pointer;
  transition: all 0.25s ease;
  border: none;
}

.control-btn i {
  font-size: 16px;
}

.refresh {
  background-color: #f8f9fa;
  color: #4a5568;
  border: 1px solid #e2e8f0;
}

.refresh:hover {
  background-color: #edf2f7;
  transform: translateY(-2px);
}

.add {
  background: linear-gradient(135deg, #42b983, #3aa876);
  color: white;
  box-shadow: 0 2px 6px rgba(66, 185, 131, 0.3);
}

.add:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 10px rgba(66, 185, 131, 0.4);
}

.delete {
  background: linear-gradient(135deg, #ff6b6b, #f56565);
  color: white;
  box-shadow: 0 2px 6px rgba(245, 101, 101, 0.3);
}

.delete:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 10px rgba(245, 101, 101, 0.4);
}

/* Custom scrollbar */
.list-wrapper::-webkit-scrollbar {
  width: 8px;
}

.list-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 10px;
}

.list-wrapper::-webkit-scrollbar-thumb {
  background: #cbd5e0;
  border-radius: 10px;
}

.list-wrapper::-webkit-scrollbar-thumb:hover {
  background: #a0aec0;
}
</style>

<script>
import ContactCard from "@/components/ContactCard.vue";
import InputSearch from "@/components/InputSearch.vue";
import ContactList from "@/components/ContactList.vue";
import ContactService from "@/services/contact.service";
export default {
  components: {
    ContactCard,
    InputSearch,
    ContactList,
  },
  data() {
    return {
      contacts: [],
      activeIndex: -1,
      searchText: "",
    };
  },
  watch: {
    // Giám sát các thay đổi của biến searchText.
    // Bỏ chọn phần tử đang được chọn trong danh sách.
    searchText() {
      this.activeIndex = -1;
    },
  },
  computed: {
    // Chuyển các đối tượng contact thành chuỗi để tiện cho tìm kiếm.
    contactStrings() {
      return this.contacts.map((contact) => {
        const { name, email, address, phone } = contact;
        return [name, email, address, phone].join("");
      });
    },
    // Trả về các contact có chứa thông tin cần tìm kiếm.
    filteredContacts() {
      if (!this.searchText) return this.contacts;
      return this.contacts.filter((_contact, index) =>
        this.contactStrings[index].includes(this.searchText)
      );
    },
    activeContact() {
      if (this.activeIndex < 0) return null;
      return this.filteredContacts[this.activeIndex];
    },
    filteredContactsCount() {
      return this.filteredContacts.length;
    },
  },
  methods: {
    async retrieveContacts() {
      try {
        this.contacts = await ContactService.getAll();
      } catch (error) {
        console.log(error);
      }
    },
    refreshList() {
      this.retrieveContacts();
      this.activeIndex = -1;
    },
    async removeAllContacts() {
      if (confirm("Bạn muốn xóa tất cả Liên hệ?")) {
        try {
          await ContactService.deleteAll();
          this.refreshList();
        } catch (error) {
          console.log(error);
        }
      }
    },
    goToAddContact() {
      this.$router.push({ name: "contact.add" });
    },
  },
  mounted() {
    this.refreshList();
  },
};
</script>