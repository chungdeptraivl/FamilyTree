<template>
  <div>
    <b-form
      v-if="show"
      class="b-form"
      enctype="multipart/form-data"
      @submit.prevent="onSubmit"
      @reset="onReset"
    >
      <div class="row">
        <!-- Trường họ và tên -->
        <b-form-radio-group
          class="col-md-6"
          :state="isNameValid"
          :invalid-feedback="nameErrorMessage"
        >
          <label
            >Họ và tên
            <span style="color: red; margin-left: 2px">*</span>
          </label>
          <b-form-input
            v-model="form.name"
            type="text"
            placeholder="Nhập tên..."
            required
            @blur="validateName"
          ></b-form-input>
        </b-form-radio-group>

        <!-- Trường chọn hình ảnh từ file -->
        <b-form-group label="Chọn ảnh đại diện:" class="col-md-6">
          <b-form-file
            placeholder="Chọn ảnh..."
            accept="image/*"
            @change="onFileChosen"
          ></b-form-file>
        </b-form-group>
      </div>

      <div class="row">
        <div class="col-md-6">
          <!-- Trường giới tính -->
          <b-form-radio-group class="mt-3" :state="form.selectedSex !== ''">
            <label
              >Giới tính
              <span style="color: red; margin-left: 2px">*</span>
            </label>
            <b-form-radio-group
              v-model="form.selectedSex"
              name="radio-sex"
              required
            >
              <b-form-radio value="true">Nam</b-form-radio>
              <b-form-radio value="false">Nữ</b-form-radio>
            </b-form-radio-group>
          </b-form-radio-group>

          <!-- Trường tình trạng -->
          <b-form-group class="mt-2" :state="form.selectedStatus !== ''">
            <label
              >Tình trạng
              <span style="color: red; margin-left: 2px">*</span>
            </label>
            <b-form-radio-group
              v-model="form.selectedStatus"
              name="radio-status"
              required
            >
              <b-form-radio value="true" @change="deleteValueDeath"
                >Còn sống</b-form-radio
              >
              <b-form-radio value="false">Từ trần</b-form-radio>
            </b-form-radio-group>
          </b-form-group>
        </div>

        <!-- Hiển thị hình ảnh -->
        <div class="col-md-6 d-flex justify-content-center">
          <img
            class="mt-2"
            width="200px"
            height="100%"
            :src="
              form.img
                ? form.img
                : 'https://icons.veryicon.com/png/o/internet--web/55-common-web-icons/person-4.png'
            "
            alt="person_image"
            style="object-fit: cover"
          />
        </div>
      </div>

      <div class="row">
        <!-- trường nghề nghiệp -->
        <b-form-group label="Nghề nghiệp:" class="col-md-6">
          <b-form-input
            v-model="form.job"
            type="text"
            placeholder="Nhập nghề nghiệp..."
          ></b-form-input>
        </b-form-group>

        <!-- Trường chọn địa chỉ -->
        <b-form-group label="Địa chỉ:" class="col-md-6">
          <b-form-input
            v-model="form.address"
            type="text"
            placeholder="Nhập địa chỉ..."
          ></b-form-input>
        </b-form-group>
      </div>

      <div class="row">
        <!-- Trường chọn ngày sinh -->
        <b-form-group
          label="Ngày sinh:"
          class="col-md-6"
          :state="isBirthdayValid"
          :invalid-feedback="birthdayErrorMessage"
        >
          <b-form-input
            v-model="form.birthday"
            type="date"
            class="mb-2"
            @input="validateDateOfBirth"
          ></b-form-input>
        </b-form-group>

        <!-- Trường chọn ngày mất -->
        <b-form-group
          v-if="form.selectedStatus === 'false'"
          label="Ngày mất:"
          class="col-md-6"
          :state="isDeathdayValid"
          :invalid-feedback="deathdayErrorMessage"
        >
          <b-form-input
            v-model="form.deathday"
            type="date"
            class="mb-2"
            @input="validateDateOfDeath"
          ></b-form-input>
        </b-form-group>
      </div>

      <!-- Sử lý các sự kiện -->
      <div class="d-flex justify-content-end">
        <b-button type="reset" class="mr-3" variant="danger">Reset</b-button>
        <b-button type="submit" variant="primary" :disabled="!validForm"
          >Thêm người</b-button
        >
      </div>
    </b-form>
  </div>
</template>

<script>
export default {
  props: {
    id: {
      type: Number,
      default: null,
    },

    item: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      form: {
        name: '',
        img: null,
        selectedSex: '',
        selectedStatus: '',
        birthday: '',
        address: '',
        job: '',
        deathday: '',
      },
      show: true,
      familyTreeId: null,
      validForm: false,
      isNameValid: false,
      nameErrorMessage: '',
      isBirthdayValid: true,
      isDeathdayValid: true,
      birthdayErrorMessage: '',
      deathdayErrorMessage: '',
    }
  },

  watch: {
    form: {
      deep: true,
      handler() {
        this.validateForm()
      },
    },
  },

  created() {
    // Lấy ID sản phẩm từ query parameter khi trang được tạo
    this.familyTreeId = this.$route.query.id
  },

  methods: {
    onReset(event) {
      event.preventDefault()
      // Reset our form values
      this.form.name = ''
      this.form.img = null
      this.form.selectedSex = ''
      this.form.selectedStatus = ''
      this.form.birthday = ''
      this.form.address = ''
      this.form.job = ''
      this.form.deathday = ''
      this.nameErrorMessage = ''
      this.birthdayErrorMessage = ''
      this.deathdayErrorMessage = ''
    },

    deleteValueDeath() {
      this.form.deathday = ''
      this.deathdayErrorMessage = ''
    },

    async onFileChosen(event) {
      const file = event.target.files[0]
      if (file) {
        const formData = new FormData()
        formData.append('image', file)

        try {
          const response = await fetch('https://api.imgur.com/3/image/', {
            method: 'POST',
            headers: {
              Authorization: 'Client-ID 59ab98495609928',
            },
            body: formData,
          })

          const data = await response.json()
          this.form.img = data.data.link // Lưu đường dẫn hình ảnh vào biến img
          // eslint-disable-next-line no-console
          console.log(this.form.img)
          // eslint-disable-next-line no-console
          console.log(data.data.link)
        } catch (error) {
          // eslint-disable-next-line no-console
          console.log(error)
        }
      }
    },

    validateForm() {
      // Kiểm tra các trường khác
      if (
        this.isBirthdayValid &&
        this.isDeathdayValid &&
        this.form.name.length > 0 &&
        this.form.name.length <= 50 &&
        this.form.selectedSex !== '' &&
        this.form.selectedStatus !== ''
      ) {
        this.validForm = true
      } else {
        this.validForm = false
      }
    },

    // Validate tên
    validateName() {
      if (this.form.name.length === 0) {
        this.isNameValid = false
        this.nameErrorMessage = 'Vui lòng nhập họ và tên.'
      } else if (this.form.name.length > 50) {
        this.isNameValid = false
        this.nameErrorMessage = 'Tên không được vượt quá 50 ký tự.'
      } else {
        this.isNameValid = true
        this.nameErrorMessage = ''
      }
      this.validateForm()
    },

    // Validate ngày sinh phải trước ngày hiện tại
    validateDateOfBirth() {
      const currentDate = new Date()
      const dateOfBirth = new Date(this.form.birthday)
      if (dateOfBirth > currentDate) {
        this.isBirthdayValid = false
        this.birthdayErrorMessage =
          'Ngày sinh không được vượt quá ngày hiện tại.'
      } else {
        this.isBirthdayValid = true
      }
      this.validateForm()
    },

    // Validate ngày mất phải trước ngày hiện tại và sau ngày sinh
    validateDateOfDeath() {
      const currentDate = new Date()
      const dateOfDeath = new Date(this.form.deathday)
      if (dateOfDeath > currentDate) {
        this.isDeathdayValid = false
        this.deathdayErrorMessage = 'Ngày mất không hợp lệ.'
      } else if (dateOfDeath < new Date(this.form.birthday)) {
        this.isDeathdayValid = false
        this.deathdayErrorMessage = 'Ngày mất phải sau ngày sinh.'
      } else {
        this.isDeathdayValid = true
      }
      this.validateForm()
    },

    showSuccessToast(message) {
      this.$bvToast.toast(message, {
        title: 'Thành công',
        variant: 'success',
        autoHideDelay: 5000,
      })
    },
    showErrorToast(message) {
      this.$bvToast.toast(message, {
        title: 'Lỗi',
        variant: 'danger',
        autoHideDelay: 5000,
      })
    },

    async onSubmit() {
      try {
        await this.$axios.$post(
          `http://localhost:8080/person/createParents?personId=${this.id}`,
          {
            personName: this.form.name,
            personGender: this.form.selectedSex,
            personDob: this.form.birthday,
            personJob: this.form.job,
            personReligion: null,
            personEthnic: null,
            personDod: this.form.deathday,
            personAddress: this.form.address,
            parentsId: null,
            // familyTreeId: this.familyTreeId,
            familyTreeId: this.item.familyTreeId,
            personStatus: this.form.selectedStatus,
            personRank: null,
            personDescription: null,
            personStory: null,
            fatherId: null,
            motherId: null,
            personImage: this.form.img,
            siblingNum: null,
            groupChildId: null,
          }
        )

        // eslint-disable-next-line no-console
        console.log('success')

        this.$emit('personCreated')

        window.location.reload()

        this.showSuccessToast('Thêm người vào phả đồ thành công')
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error)

        this.showErrorToast('Có lỗi xảy ra khi thêm người vào phả đồ!')
      }
    },
  },
}
</script>
