<template>
  
  <h4>Dynamic</h4>
  <div v-if="githubEvents.length > 0" class="github-events">
    <div v-for="event in githubEvents" :key="event.id" class="event-item mb-2">
      <div class="d-flex justify-content-between">
        <span class="badge bg-success align-self-center">{{ event.type.toUpperCase() }}</span>
        <small class="text-muted">{{ event.timeAgo }}</small>
      </div>
      <div class="event-repo mt-1">
        <strong>{{ event.repo }}</strong>
      </div>
      <div v-if="event.details" class="event-details text-muted small">
        {{ event.details }}
      </div>
    </div>
  </div>
  <div v-else class="text-muted">
    Loading GitHub activities...
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';

const helloTip = ref("");
const showInfo = ref(false);
const githubEvents = ref<Array<{
  id: string;
  type: string;
  repo: string;
  timeAgo: string;
  details?: string;
}>>([]);

// GitHub event interfaces
interface GitHubActor {
  id: number;
  login: string;
  display_login: string;
  gravatar_id: string;
  url: string;
  avatar_url: string;
}

interface GitHubRepo {
  id: number;
  name: string;
  url: string;
}

interface GitHubPayload {
  [key: string]: any;
}

interface GitHubEvent {
  id: string;
  type: string;
  actor: GitHubActor;
  repo: GitHubRepo;
  payload: GitHubPayload;
  public: boolean;
  created_at: string;
}

function formatTimeAgo(dateString: string): string {
  const now = new Date();
  const eventDate = new Date(dateString);
  const diffInSeconds = Math.floor((now.getTime() - eventDate.getTime()) / 1000);
  
  if (diffInSeconds < 60) {
    return `${diffInSeconds} seconds ago`;
  } else if (diffInSeconds < 3600) {
    const minutes = Math.floor(diffInSeconds / 60);
    return `${minutes} minute${minutes > 1 ? 's' : ''} ago`;
  } else if (diffInSeconds < 86400) {
    const hours = Math.floor(diffInSeconds / 3600);
    return `${hours} hour${hours > 1 ? 's' : ''} ago`;
  } else {
    const days = Math.floor(diffInSeconds / 86400);
    return `${days} day${days > 1 ? 's' : ''} ago`;
  }
}

function getEventDescription(event: GitHubEvent): { type: string; details?: string } {
  switch (event.type) {
    case 'PushEvent':
      const commits = event.payload.commits || [];
      const commitCount = commits.length;
      if (commitCount > 0) {
        const firstCommit = commits[0];
        return {
          type: 'Pushed',
          details: `${commitCount} commit${commitCount > 1 ? 's' : ''}: ${firstCommit.message.split('\n')[0]}`
        };
      }
      return { type: 'Pushed', details: `${commitCount} commit${commitCount > 1 ? 's' : ''}` };
    
    case 'WatchEvent':
      return { type: 'Starred' };
    
    case 'IssuesEvent':
      return { type: `${event.payload.action} issue`, details: event.payload.issue?.title };
    
    case 'PullRequestEvent':
      return { type: `${event.payload.action} pull request`, details: event.payload.pull_request?.title };
    
    case 'ForkEvent':
      return { type: 'Forked' };
    
    case 'CreateEvent':
      return { type: 'Created', details: `${event.payload.ref_type} ${event.payload.ref}` };
    
    default:
      return { type: event.type.replace('Event', '') };
  }
}

function setHelloTip(text: string, next_action: undefined | VoidFunction = undefined) {
    const interval = setInterval(() => {
    if (helloTip.value.length < text.length) {
        helloTip.value = text.substring(0, helloTip.value.length + 1);
    } else {
        clearInterval(interval);
        if (next_action !== undefined) {
            next_action();
        }
    }
    }, 37);
}

onMounted(async () => {
    showInfo.value = false;
    setHelloTip("👋 Hi, there!", () => {
        setTimeout(() => {
            helloTip.value = ""
            document.getElementById("hello_tip")?.classList.add("h2");
            setHelloTip("I'm XiaoDeng3386.", () => {
                showInfo.value = true;
            });
        }, 1200);
    });
    await getGithubFeed()
})

async function getGithubFeed() {
  try {
    const response = await fetch('https://api.github.com/users/This-is-XiaoDeng/events/public');
    if (!response.ok) {
      throw new Error(`GitHub API request failed with status ${response.status}`);
    }
    
    const events: GitHubEvent[] = await response.json();
    
    // Process events for display
    const processedEvents = events.slice(0, 5).map(event => {
      const description = getEventDescription(event);
      return {
        id: event.id,
        type: description.type,
        repo: event.repo.name,
        timeAgo: formatTimeAgo(event.created_at),
        details: description.details
      };
    });
    
    githubEvents.value = processedEvents;
  } catch (error) {
    console.error('Error fetching GitHub events:', error);
    // Set some default events or error message if needed
    githubEvents.value = [];
  }
}

</script>

<style scoped>
.github-events {
  max-height: 300px;
  overflow-y: auto;
  padding-right: 10px;
}

.event-item {
  padding: 10px;
  border-radius: 5px;
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.event-item:not(:last-child) {
  margin-bottom: 10px;
}

.event-repo {
  font-size: 0.9em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.event-details {
  margin-top: 5px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Scrollbar styling */
.github-events::-webkit-scrollbar {
  width: 6px;
}

.github-events::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.github-events::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.github-events::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}
</style>
