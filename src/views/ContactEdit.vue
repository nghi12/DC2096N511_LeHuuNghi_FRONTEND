<template>
  <div v-if="contact" class="page">
    <h4>Hiệu chỉnh Liên hệ</h4>
    <ContactForm
      :contact="contact"
      @submit:contact="updateContact"
      @delete:contact="deleteContact"
    />
    <p>{{ message }}</p>
  </div>
</template>
<script>
import ContactForm from "@/components/ContactForm.vue";
import ContactService from "@/services/contact.service";

export default {
  components: {
    ContactForm,
  },
  props: {
    id: { type: String, required: true },
  },
  data() {
    return {
      contact: null,
      message: "",
    };
  },
  methods: {
    async getContact(id) {
      try {
        this.contact = await ContactService.get(id);
      } catch (error) {
        console.error(error);
        this.$router.push({
          name: "notfound",
          params: {
            pathMatch: this.$route.path.split("/").slice(1),
          },
          query: this.$route.query,
          hash: this.$route.hash,
        });
      }
    },
    async updateContact(data) {
      try {
        console.log(
          "Updating contact ID:",
          this.contact._id,
          "with data:",
          data
        );
        const response = await ContactService.update(this.contact._id, data);
        alert("Liên hệ được cập nhật thành công");
        this.$router.push({ name: "contactbook" });
      } catch (error) {
        console.error(error);
      }
    },
    async deleteContact() {
      if (confirm("Ban muon xoa lien he nay?")) {
        try {
          await ContactService.delete(this.contact._id);
          this.$router.push({ name: "contactbook" });
        } catch (error) {
          console.error(error);
        }
      }
    },
  },
  created() {
    this.getContact(this.id);
    this.message = "";
  },
};
</script>