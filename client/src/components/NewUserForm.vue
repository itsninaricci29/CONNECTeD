<template>
  <div>
    <div>
      <h1>CONNECTeD</h1>
      <h6>Complete the form.</h6>
    </div>
    <div class="form" method="submit">
      <div class="form-group">
        <label>First Name</label>
        <span class="star">*</span>

        <input
          type="First Name"
          class="form-control active"
          placeholder="John"
          v-model="firstname"
        />
      </div>
      <div class="form-group">
        <label>Last Name</label>
        <span class="star">*</span>

        <input
          type="Last Name"
          class="form-control active"
          placeholder="Smith"
          v-model="lastname"
        />
      </div>
      <div class="form-group">
        <label>Year of Study</label>
        <span class="star">*</span>

        <select class="form-control form-control-md" v-model="yos">
          <option value="" selected>1</option>
          <option>2</option>
          <option>3</option>
          <option>4</option>
          <option>4+</option>
        </select>
      </div>
      <div class="form-group">
        <label>Major</label>
        <span class="star">*</span>

        <select class="form-control form-control-md" v-model="major">
          <option value="" disabled selected>select your major</option>
          <option>Computer Science</option>
          <option>Mathematics</option>
          <option>Statistics</option>
        </select>
      </div>
      <div class="form-group">
        <label>Bio</label>
        <span class="star">*</span>

        <input
          type="bio"
          class="form-control active"
          placeholder="Got a project? Let's collaborate!"
          maxlength="50"
          v-model="bio"
        />
        <small id="bio-type" class="form-text text-muted"
          >Describe your self!</small
        >
      </div>

      <div class="form-group">
        <label>Website</label>

        <input
          type="website"
          class="form-control active"
          placeholder="https://myWebsite.ca"
          maxlength="100"
          v-model="website"
        />
        <small id="website-type" class="form-text text-muted"
          >Link your Github!</small
        >
      </div>
      <div>
        <b-form-group label="select your tags:">
          <!-- prop `add-on-change` is needed to enable adding tags vie the `change` event -->
          <b-form-tags
            v-model="value"
            size="sm"
            add-on-change
            no-outer-focus
            class="mb-2 outer"
          >
            <template
              v-slot="{ tags, inputAttrs, inputHandlers, disabled, removeTag }"
            >
              <ul v-if="tags.length > 0" class="list-inline">
                <li v-for="tag in tags" :key="tag" class="abc">
                  <b-form-tag
                    @remove="removeTag(tag)"
                    :title="tag"
                    :disabled="disabled"
                    variant="info"
                    >{{ tag }}</b-form-tag
                  >
                </li>
              </ul>
              <b-form-select
                class="form-control abc"
                v-bind="inputAttrs"
                v-on="inputHandlers"
                :disabled="disabled || availableOptions.length === 0"
                :options="availableOptions"
              >
                <template v-slot:first>
                  <!-- This is required to prevent bugs with Safari -->
                  <option disabled value="">Choose a tag...</option>
                </template>
              </b-form-select>
            </template>
          </b-form-tags>
        </b-form-group>
      </div>
      <div class="form-group">
        <label>Profile Picture </label>
        <br />
        <input
          type="file"
          id="file"
          ref="file"
          v-on:change="handleFileUpload()"
        />
      </div>

      <div id="button">
        <p class="star">{{ error }}</p>
        <button
          type="log-in-via-utorid"
          class="btn btn-primary"
          @click="submit"
        >
          SUBMIT
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "submit",
  data() {
    return {
      firstname: "",
      lastname: "",
      bio: "",
      website: "",
      yos: "",
      major: "",
      file: "",
      error: "",
      tags: null,
      options: [],
      value: []
    };
  },
  computed: {
    availableOptions() {
      return this.options.filter(opt => this.value.indexOf(opt) === -1);
    }
  },
  mounted() {
    axios.get("/connect/tags").then(response => {
      this.tags = response.data;
      for (var i = 0; i < this.tags.length; i++) {
        this.options.push(this.tags[i].tag_name);
      }
    });
  },
  methods: {
    submit() {
      if(this.firstname == '' || this.lastname == ''){
        this.error = "Please enter your full name"
        return
      }else if(this.major == ''){
        this.error = "Please enter a major"
        return
      }else if(this.bio == ''){
        this.error = "Please enter a bio"
        return
      }
      else if(this.value.length == 0){
        this.error = "Please select at least one tag"
        return
      }
      var ids = [];
      for (var i = 0; i < this.value.length; i++) {
        for (var j = 0; j < this.tags.length; j++) {
          if (this.value[i] == this.tags[j].tag_name) {
            ids.push(this.tags[j].id);
          }
        }
      }

      var form = new FormData();
      form.append("utorid", this.$store.state.user.utorid);
      form.append("first_name", this.firstname);
      form.append("last_name", this.lastname);
      form.append("bio", this.bio);
      form.append("major", this.major);
      form.append("year", this.yos);
      form.append("website", this.website);
      form.append("profile_picture", this.file);
      form.append("tag_ids", JSON.stringify(ids));
      axios
        .post("/connect/create_profile", form, {
          headers: { "Content-Type": "multipart/form-data" }
        })
        .then(response => {
          (this.firstname = response.data.first_name),
            (this.lastname = response.data.last_name),
            (this.bio = response.data.bio);
          this.yos = response.data.year;
          this.major = response.data.major;
          this.website = response.data.website;
          this.$cookies.set('id', response.data.user.id)
          this.$router.push({ path: `/profile/${response.data.user.id}` });
        })
        .catch(() => {
          this.error = "Something went wrong, please try again shortly.";
        });
    },
    handleFileUpload() {
      this.error = "";
      var file = this.$refs.file.files[0];
      var size = file.size / 1024 / 1024; // in MB
      if (size > 2) {
        this.error = "Please select a file under 2MB";
      } else if (file.type != "image/jpeg" && file.type != "image/png") {
        this.error = "Please select a png or jpg image";
      } else {
        this.file = file;
      }
    }
  }
};
</script>

<style scoped>
body {
  padding-top: 40px;
  background-color: white;
  color: black;
  text-align: center;
}
body h1 {
  font-weight: bolder;
  font-size: 60px;
  text-align: center;
}
body h6 {
  font-weight: bold;
  font-size: 20px;
  padding-top: 10px;
}
.form {
  display: inline-block;
}
.form-group {
  text-align: left;
}
.form-control {
  width: 500px;
  border-color: black;
}
.active {
  color: black;
}
#bio-type {
  width: 500px;
}
#button {
  padding-bottom: 20px;
  padding-top: 20px;
}
.btn {
  background-color: #2e2e2e;
}
.btn-primary {
  border-color: white;
  background-color: #2e2e2e;
}
.btn:hover {
  background-color: white;
  color: black;
  border-color: black;
}
.star {
  color: red;
}

.abc {
  display: inline;
}

.abc:not(:first-child) {
  padding-left: 10px;
}

.outer {
  border-color: white;
  padding: 0;
}

.outer span {
  background-color: lightslategrey;
  border-color: black;
  color: white;
}
</style>
