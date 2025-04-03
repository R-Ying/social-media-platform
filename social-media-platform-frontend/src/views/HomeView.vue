<template>
  <div class="home-container">
    <header class="header">
      <h1>社群媒體平台</h1>
      <div class="user-info">
        <span>{{ currentUser }}</span>
        <button class="btn btn-outline" @click="logout">登出</button>
      </div>
    </header>

    <!-- 發文區塊 -->
    <div class="post-creator">
      <h2>分享新貼文</h2>
      <div class="post-form">
        <textarea
          v-model="newPostContent"
          placeholder="有什麼新鮮事想分享嗎？"
          maxlength="500"
        ></textarea>
        <div class="action-bar">
          <span>{{ 500 - newPostContent.length }} 字元</span>
          <button
            class="btn btn-primary"
            :disabled="!newPostContent.trim()"
            @click="createPost"
          >
            發佈
          </button>
        </div>
      </div>
    </div>

    <!-- 貼文列表 -->
    <div class="posts-container">
      <div v-for="post in posts" :key="post.id" class="post-card">
        <div class="post-header">
          <div class="author-info">
            <div class="avatar">{{ post.author.charAt(0) }}</div>
            <div>
              <h3>{{ post.author }}</h3>
              <span class="post-time">{{ formatDate(post.createdAt) }}</span>
              <span v-if="post.updatedAt" class="post-edited">(已編輯)</span>
            </div>
          </div>
          <div v-if="post.author === currentUser" class="post-actions">
            <button class="action-btn" @click="startEditPost(post)">編輯</button>
            <button class="action-btn delete" @click="confirmDeletePost(post.id)">刪除</button>
          </div>
        </div>

        <!-- 編輯模式 -->
        <div v-if="editingPostId === post.id" class="edit-form">
          <textarea
            v-model="editPostContent"
            maxlength="500"
          ></textarea>
          <div class="action-bar">
            <button class="btn btn-outline" @click="cancelEdit">取消</button>
            <button
              class="btn btn-primary"
              :disabled="!editPostContent.trim()"
              @click="updatePost(post.id)"
            >
              更新
            </button>
          </div>
        </div>

        <!-- 貼文內容 -->
        <div v-else class="post-content">
          {{ post.content }}
        </div>

        <div class="post-footer">
          <button class="footer-btn">
            <span class="like-icon">♥</span> {{ post.likes }}
          </button>
          <button class="footer-btn" @click="toggleComments(post.id)">
            留言 ({{ post.comments.length }})
          </button>
        </div>

        <!-- 留言區塊 -->
        <div v-if="expandedPostId === post.id" class="comments-section">
          <div class="comments-list">
            <div v-for="comment in post.comments" :key="comment.id" class="comment">
              <div class="comment-header">
                <div class="comment-avatar">{{ comment.author.charAt(0) }}</div>
                <strong>{{ comment.author }}</strong>
                <span class="comment-time">{{ formatDate(comment.createdAt) }}</span>
              </div>
              <div class="comment-content">
                {{ comment.content }}
              </div>
            </div>
          </div>

          <div class="comment-form">
            <textarea
              v-model="newCommentContent[post.id]"
              placeholder="留下您的評論..."
              maxlength="200"
            ></textarea>
            <button
              class="btn btn-primary"
              :disabled="!newCommentContent[post.id]?.trim()"
              @click="addComment(post.id)"
            >
              送出
            </button>
          </div>
        </div>
      </div>

      <!-- 無貼文時顯示 -->
      <div v-if="posts.length === 0" class="no-posts">
        尚無貼文。成為第一個發文的人吧！
      </div>
    </div>

    <!-- 刪除確認對話框 -->
    <div v-if="showDeleteConfirm" class="modal-overlay">
      <div class="modal-dialog">
        <h3>確認刪除</h3>
        <p>您確定要刪除這則貼文嗎？此操作無法復原。</p>
        <div class="modal-actions">
          <button class="btn btn-outline" @click="cancelDelete">取消</button>
          <button class="btn btn-danger" @click="deletePost">刪除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';

interface Post {
  id: number;
  author: string;
  content: string;
  createdAt: Date;
  updatedAt: Date | null;
  likes: number;
  comments: Comment[];
}

interface Comment {
  id: number;
  postId: number;
  author: string;
  content: string;
  createdAt: Date;
}

// 當前使用者
const currentUser = ref('小明');

// 貼文資料
const posts = ref<Post[]>([
  {
    id: 1,
    author: '小明',
    content: '今天天氣真好，出去走走吧！',
    createdAt: new Date(2025, 3, 2, 10, 30),
    updatedAt: null,
    likes: 5,
    comments: [
      {
        id: 1,
        postId: 1,
        author: '小花',
        content: '確實是個好天氣！',
        createdAt: new Date(2025, 3, 2, 10, 45)
      }
    ]
  },
  {
    id: 2,
    author: '小花',
    content: '推薦一本好書：《深入淺出 Vue.js》',
    createdAt: new Date(2025, 3, 1, 15, 20),
    updatedAt: null,
    likes: 8,
    comments: []
  }
]);

// 新增貼文相關
const newPostContent = ref('');

// 編輯貼文相關
const editingPostId = ref<number | null>(null);
const editPostContent = ref('');

// 刪除貼文相關
const showDeleteConfirm = ref(false);
const postToDeleteId = ref<number | null>(null);

// 留言相關
const expandedPostId = ref<number | null>(null);
const newCommentContent = reactive<Record<number, string>>({});

const router = useRouter();

// 格式化日期
const formatDate = (date: Date): string => {
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffMins = Math.floor(diffMs / 60000);
  const diffHours = Math.floor(diffMins / 60);
  const diffDays = Math.floor(diffHours / 24);

  if (diffMins < 1) return '剛剛';
  if (diffMins < 60) return `${diffMins}分鐘前`;
  if (diffHours < 24) return `${diffHours}小時前`;
  if (diffDays < 7) return `${diffDays}天前`;

  return `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
};

// 新增貼文
const createPost = () => {
  if (!newPostContent.value.trim()) return;

  const newPost: Post = {
    id: Date.now(), // 用時間戳作為ID
    author: currentUser.value,
    content: newPostContent.value,
    createdAt: new Date(),
    updatedAt: null,
    likes: 0,
    comments: []
  };

  posts.value.unshift(newPost);
  newPostContent.value = '';
};

// 開始編輯貼文
const startEditPost = (post: Post) => {
  editingPostId.value = post.id;
  editPostContent.value = post.content;
};

// 更新貼文
const updatePost = (id: number) => {
  if (!editPostContent.value.trim()) return;

  const index = posts.value.findIndex(post => post.id === id);
  if (index !== -1) {
    posts.value[index].content = editPostContent.value;
    posts.value[index].updatedAt = new Date();
    editingPostId.value = null;
  }
};

// 取消編輯
const cancelEdit = () => {
  editingPostId.value = null;
};

// 確認刪除貼文
const confirmDeletePost = (id: number) => {
  postToDeleteId.value = id;
  showDeleteConfirm.value = true;
};

// 刪除貼文
const deletePost = () => {
  if (postToDeleteId.value) {
    posts.value = posts.value.filter(post => post.id !== postToDeleteId.value);
    postToDeleteId.value = null;
    showDeleteConfirm.value = false;
  }
};

// 取消刪除
const cancelDelete = () => {
  postToDeleteId.value = null;
  showDeleteConfirm.value = false;
};

// 切換顯示留言
const toggleComments = (id: number) => {
  expandedPostId.value = expandedPostId.value === id ? null : id;
};

// 新增留言
const addComment = (postId: number) => {
  const content = newCommentContent[postId];
  if (!content?.trim()) return;

  const index = posts.value.findIndex(post => post.id === postId);
  if (index !== -1) {
    const newComment: Comment = {
      id: Date.now(), // 用時間戳作為ID
      postId,
      author: currentUser.value,
      content,
      createdAt: new Date()
    };

    posts.value[index].comments.push(newComment);
    newCommentContent[postId] = '';
  }
};

const logout = () => {
  router.push('/login');
};
</script>

<style scoped>
.home-container {
  width: 100%;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  font-family: 'Arial', sans-serif;
  background-color: #1e1e1e;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 20px;
  margin-bottom: 20px;
  border-bottom: 1px solid #333;
  background-color: #1e1e1e;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.post-creator {
  background: #2a2a2a;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  width: 600px;
  max-width: 90%;
}

.post-form, .edit-form {
  display: flex;
  flex-direction: column;
}

textarea {
  width: 100%;
  min-height: 100px;
  padding: 12px;
  border: 1px solid #444;
  border-radius: 4px;
  resize: vertical;
  font-family: inherit;
  margin-bottom: 10px;
  background-color: #333;
  color: white;
}

.action-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn {
  padding: 8px 16px;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
}

.btn-primary {
  background-color: #4b70e2;
  color: white;
}

.btn-primary:hover {
  background-color: #3a5bbf;
}

.btn-primary:disabled {
  background-color: #a0b1e0;
  cursor: not-allowed;
}

.btn-outline {
  background-color: transparent;
  border: 1px solid #4b70e2;
  color: #4b70e2;
}

.btn-outline:hover {
  background-color: rgba(75, 112, 226, 0.1);
}

.btn-danger {
  background-color: #e24b4b;
  color: white;
}

.btn-danger:hover {
  background-color: #bf3a3a;
}

.posts-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 600px;
  max-width: 90%;
}

.post-card {
  background: #2a2a2a;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  width: 100%;
}

.post-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.author-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #4b70e2;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

.post-time, .comment-time, .post-edited {
  color: #aaa;
  font-size: 0.85em;
}

.post-actions {
  display: flex;
  gap: 10px;
}

.action-btn {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  font-size: 0.9em;
}

.action-btn:hover {
  text-decoration: underline;
}

.action-btn.delete:hover {
  color: #e24b4b;
}

.post-content {
  margin-bottom: 15px;
  line-height: 1.5;
  white-space: pre-wrap;
}

.post-footer {
  display: flex;
  gap: 15px;
  border-top: 1px solid #444;
  padding-top: 15px;
}

.footer-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  background: none;
  border: none;
  color: #555;
  cursor: pointer;
  font-size: 0.9em;
}

.footer-btn:hover {
  color: #4b70e2;
}

.like-icon {
  color: #e24b4b;
}

.comments-section {
  margin-top: 15px;
  border-top: 1px solid #444;
  padding-top: 15px;
}

.comments-list {
  margin-bottom: 15px;
}

.comment {
  padding: 10px;
  border-radius: 4px;
  background: #333;
  margin-bottom: 10px;
}

.comment-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 5px;
}

.comment-avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #4b70e2;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85em;
  font-weight: bold;
}

.comment-content {
  margin-left: 38px;
  line-height: 1.4;
}

.comment-form {
  display: flex;
  gap: 10px;
  margin-top: 15px;
}

.comment-form textarea {
  min-height: 60px;
  flex: 1;
}

.no-posts {
  text-align: center;
  padding: 40px;
  color: #777;
  font-style: italic;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-dialog {
  background: #2a2a2a;
  border-radius: 8px;
  padding: 20px;
  width: 400px;
  max-width: 90%;
  color: white;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}
</style>
