<template>
  <div>
    <button
      type="button"
      class="btn btn-primary float-right"
      data-toggle="modal"
      data-target="#myModal"
      @click="clearForm"
    >Add New Article</button>

    <!-- The Modal -->
    <div class="modal" id="myModal">
      <div class="modal-dialog">
        <div class="modal-content">
          <!-- Modal header -->
          <div class="modal-header">
            <h5 class="modal-title">Article Manager</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <!-- Modal body -->
          <div class="modal-body">
            <form @submit.prevent="addArticle">
              <div class="form-group">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Title"
                  v-model="article.title"
                  required="required"
                />
              </div>
              <div class="form-group">
                <textarea
                  class="form-control"
                  placeholder="Body"
                  v-model="article.body"
                  style="height:200px;"
                  required="required"
                ></textarea>
              </div>
              <button class="btn btn-success btn-block mb-3" type="submit">Save</button>
            </form>
          </div>
          <!-- Modal footer -->
          <div class="modal-footer">
            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item">
          <a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a>
        </li>
        <li class="page-item disabled">
          <a
            class="page-link text-dark"
            href="#"
          >Page {{ pagination.current_page }} of {{ pagination.last_page }}</a>
        </li>
        <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item">
          <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
        </li>
      </ul>
    </nav>

    <!-- Article container -->
    <div class="card card-body mb-3" v-for="article in articles" v-bind:key="article.id">
      <h3>{{ article.title }}</h3>
      <p>{{ article.body }}</p>
      <label>
        <span class="text-muted">Date Posted:</span>
        {{ formatDate(article.created_at) }}
      </label>
      <hr />
      <div class="row">
        <div class="col-md-6">
          <button
            class="btn btn-secondary btn-block mb-3"
            @click="editArticle(article)"
            data-toggle="modal"
            data-target="#myModal"
          >Edit</button>
        </div>
        <div class="col-md-6">
          <button class="btn btn-danger btn-block" @click="deleteArticle(article.id)">Delete</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      articles: [],
      article: {
        id: "",
        title: "",
        body: "",
        created_at: ""
      },
      article_id: "",
      pagination: {},
      edit: false
    };
  },
  created() {
    this.fetchArticles();
  },
  methods: {
    fetchArticles(page_url) {
      let vm = this;
      page_url = page_url || "/api/articles";
      fetch(page_url)
        .then(res => res.json())
        .then(res => {
          this.articles = res.data;
          vm.makePagination(res.meta, res.links);
        })
        .catch(err => console.log(err));
    },
    makePagination(meta, links) {
      let pagination = {
        current_page: meta.current_page,
        last_page: meta.last_page,
        next_page_url: links.next,
        prev_page_url: links.prev
      };
      this.pagination = pagination;
    },
    deleteArticle(id) {
      if (confirm("Are you sure you want to delete?")) {
        fetch(`api/article/${id}`, {
          method: "delete"
        })
          .then(res => res.json())
          .then(data => {
            this.fetchArticles();
          })
          .catch(err => console.log(err));
      }
    },
    addArticle() {
      if (this.edit == false) {
        //Add
        fetch("api/article", {
          method: "post",
          body: JSON.stringify(this.article),
          headers: {
            "content-type": "application/json"
          }
        })
          .then(res => res.json())
          .then(data => {
            this.article.title = "";
            this.article.body = "";
            this.fetchArticles();
            this.removeModal();
          })
          .catch(err => console.log(err));
      } else {
        //Update
        fetch("api/article", {
          method: "put",
          body: JSON.stringify(this.article),
          headers: {
            "content-type": "application/json"
          }
        })
          .then(res => res.json())
          .then(data => {
            this.fetchArticles();
            window.location.reload();
            this.removeModal();
          })
          .catch(err => console.log(err));
      }
    },
    editArticle(article) {
      this.edit = true;
      this.article.id = article.id;
      this.article.article_id = article.id;
      this.article.title = article.title;
      this.article.body = article.body;
    },
    formatDate(input) {
      const dateObj = new Date(input);

      const year = dateObj.getFullYear();
      const month = (dateObj.getMonth() + 1).toString().padStart(2, "0");
      const date = dateObj
        .getDate()
        .toString()
        .padStart(2, "0");

      return `${year}-${month}-${date}`;
    },
    clearForm() {
      this.article.title = "";
      this.article.body = "";
    },
    removeModal() {
      $("#myModal").modal("hide");
      $("body").removeClass("modal-open");
      $(".modal-backdrop").remove();
    }
  }
};
</script>