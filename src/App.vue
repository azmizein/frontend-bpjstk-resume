<script setup>
import { ref, computed, onMounted } from 'vue'
import RichTextEditor from './components/RichTextEditor.vue'

const profilesList = ref([])

onMounted(async () => {
  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/profile`);
    const data = await res.json();
    if (data && data.data) {
      profilesList.value = data.data;
    }
  } catch(e) {
    console.error('Failed to load profiles list');
  }
})

const notification = ref({
  show: false,
  message: '',
  type: 'success'
})

const showNotification = (msg, type = 'success') => {
  notification.value.message = msg;
  notification.value.type = type;
  notification.value.show = true;
  setTimeout(() => {
    notification.value.show = false;
  }, 3000);
}

const confirmModal = ref({
  show: false,
  message: '',
  onConfirm: null
})

const showConfirm = (msg, onConfirmCallback) => {
  confirmModal.value.message = msg;
  confirmModal.value.onConfirm = () => {
    onConfirmCallback();
    confirmModal.value.show = false;
  };
  confirmModal.value.show = true;
}

const cancelConfirm = () => {
  confirmModal.value.show = false;
}

const formData = ref({
  jobTitle: 'Software Engineer',
  firstName: '',
  lastName: '',
  email: '',
  phone: '',
  country: '',
  city: '',
  address: '',
  postalCode: '',
  drivingLicense: '',
  nationality: '',
  placeOfBirth: '',
  dateOfBirth: '',
  summary: ''
})

const currentProfileCode = ref(null)

const saveProfile = async () => {
  try {
    const method = currentProfileCode.value ? 'PUT' : 'POST';
    const url = currentProfileCode.value 
      ? `${import.meta.env.VITE_API_URL}/api/profile/${currentProfileCode.value}`
      : `${import.meta.env.VITE_API_URL}/api/profile`;
      
    const payload = {
      wantedJobTitle: formData.value.jobTitle,
      firstName: formData.value.firstName,
      lastName: formData.value.lastName,
      email: formData.value.email,
      phone: formData.value.phone,
      country: formData.value.country,
      city: formData.value.city,
      address: formData.value.address,
      postalCode: formData.value.postalCode,
      drivingLicense: formData.value.drivingLicense,
      nationality: formData.value.nationality,
      placeOfBirth: formData.value.placeOfBirth,
      dateOfBirth: formData.value.dateOfBirth
    };

    const res = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
    
    const data = await res.json();
    if (!data.error && data.profileCode) {
      currentProfileCode.value = data.profileCode;
      showNotification(`Profile saved successfully! Your Profile Code is: ${data.profileCode}`);
    } else {
      showNotification(`Failed to save: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification('Error connecting to the server');
    console.error(err);
  }
}

const saveSummary = async () => {
  if (!currentProfileCode.value) {
    showNotification('Please save Personal Details first to generate an ID!', 'warning');
    return;
  }
  
  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/working-experience/${currentProfileCode.value}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        workingExperience: formData.value.summary
      })
    });
    
    const data = await res.json();
    if (!data.error) {
      showNotification('Professional Summary saved successfully!');
    } else {
      showNotification(`Failed to save summary: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification('Error connecting to the server');
    console.error(err);
  }
}

const loadProfileCode = ref('')

const loadProfile = async () => {
  if (!loadProfileCode.value) {
    showNotification('Please select a profile to load.', 'warning');
    return;
  }
  
  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/profile/${loadProfileCode.value}`);
    if (!res.ok) {
      throw new Error('Profile not found');
    }
    
    const data = await res.json();
    if (!data.error && data.profileCode) {
      currentProfileCode.value = data.profileCode;
      formData.value.jobTitle = data.wantedJobTitle || '';
      formData.value.firstName = data.firstName || '';
      formData.value.lastName = data.lastName || '';
      formData.value.email = data.email || '';
      formData.value.phone = data.phone || '';
      formData.value.country = data.country || '';
      formData.value.city = data.city || '';
      formData.value.address = data.address || '';
      formData.value.postalCode = data.postalCode || '';
      formData.value.drivingLicense = data.drivingLicense || '';
      formData.value.nationality = data.nationality || '';
      formData.value.placeOfBirth = data.placeOfBirth || '';
      formData.value.dateOfBirth = data.dateOfBirth || '';
      
      // Load summary
      const summaryRes = await fetch(`${import.meta.env.VITE_API_URL}/api/working-experience/${loadProfileCode.value}`);
      if (summaryRes.ok) {
        const summaryData = await summaryRes.json();
        if (summaryData && summaryData.workingExperience) {
          formData.value.summary = summaryData.workingExperience;
        }
      }

      // Load photo
      try {
        const photoRes = await fetch(`${import.meta.env.VITE_API_URL}/api/photo/${loadProfileCode.value}`);
        if (photoRes.ok) {
          const photoText = await photoRes.text();
          if (photoText && photoText.startsWith('data:image')) {
            photoDataUrl.value = photoText;
          } else {
            photoDataUrl.value = null;
          }
        } else {
          photoDataUrl.value = null;
        }
      } catch (e) {
        photoDataUrl.value = null;
      }
      
      // Load employments
      try {
        const empRes = await fetch(`${import.meta.env.VITE_API_URL}/api/employment/${loadProfileCode.value}`);
        if (empRes.ok) {
          const empData = await empRes.json();
          if (empData && empData.data) {
            employments.value = empData.data.map(emp => ({
              ...emp,
              isOpen: false
            }));
          }
        }
      } catch (e) {
        console.error('Error loading employments:', e);
      }
      
      // Load educations
      try {
        const eduRes = await fetch(`${import.meta.env.VITE_API_URL}/api/education/${loadProfileCode.value}`);
        if (eduRes.ok) {
          const eduData = await eduRes.json();
          if (eduData && eduData.data) {
            educations.value = eduData.data.map(edu => ({
              ...edu,
              isOpen: false
            }));
          }
        }
      } catch (e) {
        console.error('Error loading educations:', e);
      }
      
      // Load skills
      try {
        const skillRes = await fetch(`${import.meta.env.VITE_API_URL}/api/skill/${loadProfileCode.value}`);
        if (skillRes.ok) {
          const skillData = await skillRes.json();
          if (skillData && skillData.data) {
            userSkills.value = skillData.data.map(sk => ({
              id: sk.id,
              name: sk.skill,
              levelIndex: skillLevels.indexOf(sk.level) > -1 ? skillLevels.indexOf(sk.level) : 3,
              isOpen: false
            }));
          }
        }
      } catch (e) {
        console.error('Error loading skills:', e);
      }
      
      showNotification('Profile loaded successfully!');
    } else {
      showNotification(`Failed to load: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification(err.message === 'Profile not found' ? 'Profile not found' : 'Error connecting to the server');
    console.error(err);
  }
}

const photoDataUrl = ref(null);
const fileInputRef = ref(null);

const triggerPhotoUpload = (e) => {
  e.preventDefault();
  if (!currentProfileCode.value) {
    showNotification('Please save your Personal Details first to generate an ID before uploading a photo!', 'warning');
    return;
  }
  fileInputRef.value.click();
}

const handlePhotoUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = async (e) => {
    const base64img = e.target.result;
    
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/photo/${currentProfileCode.value}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ base64img })
      });
      const data = await res.json();
      if (!data.error) {
        photoDataUrl.value = base64img;
        showNotification('Photo uploaded successfully!');
      } else {
        showNotification(`Failed to upload photo: ${data.message || 'Unknown error'}`);
      }
    } catch (err) {
      showNotification('Error connecting to upload photo');
      console.error(err);
    }
  };
  reader.readAsDataURL(file);
}

const deletePhoto = async (e) => {
  e.preventDefault();
  if (!currentProfileCode.value || !photoDataUrl.value) return;
  
  showConfirm('Are you sure you want to delete this photo?', async () => {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/photo/${currentProfileCode.value}`, {
        method: 'DELETE'
      });
      if (res.ok) {
        photoDataUrl.value = null;
        showNotification('Photo deleted successfully!');
      }
    } catch (err) {
      showNotification('Error connecting to delete photo', 'error');
    }
  });
}

const downloadPhoto = (e) => {
  e.preventDefault();
  if (!photoDataUrl.value) return;
  
  const a = document.createElement('a');
  a.href = photoDataUrl.value;
  a.download = `profile_photo_${currentProfileCode.value}.png`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
}

const saveEmploymentToDb = async (index) => {
  if (!currentProfileCode.value) {
    showNotification('Please save Personal Details first to generate an ID!', 'warning');
    return;
  }
  
  const job = employments.value[index];
  
  if (!job.jobTitle || !job.employer) {
    showNotification('Please fill in Job title and Employer before saving.', 'warning');
    return;
  }
  
  const isNew = job.id > 1000000; // Since Date.now() is huge, real DB IDs are small
  
  try {
    const url = isNew 
      ? `${import.meta.env.VITE_API_URL}/api/employment/${currentProfileCode.value}`
      : `${import.meta.env.VITE_API_URL}/api/employment/edit/${job.id}`;
      
    const res = await fetch(url, {
      method: isNew ? 'POST' : 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        jobTitle: job.jobTitle,
        employer: job.employer,
        startDate: job.startDate,
        endDate: job.endDate,
        city: job.city,
        description: job.description
      })
    });
    
    const data = await res.json();
    if (!data.error && (data.id || data.success)) {
      if (isNew && data.id) {
        job.id = data.id; // Update with real DB ID
      }
      job.isOpen = false;
      showNotification('Employment saved successfully!');
    } else {
      showNotification(`Failed to save employment: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification('Error connecting to save employment');
  }
}

const deleteEmployment = async (index) => {
  const job = employments.value[index];
  
  // If it's a new entry (not saved to DB), just remove from UI
  if (job.id > 1000000) {
    employments.value.splice(index, 1);
    return;
  }
  
  showConfirm('Are you sure you want to delete this employment?', async () => {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/employment/${currentProfileCode.value}?id=${job.id}`, {
        method: 'DELETE'
      });
      const data = await res.json();
      if (res.ok && data.profileCode) {
        employments.value.splice(index, 1);
        showNotification('Employment deleted successfully!');
      } else {
        showNotification(`Failed to delete: ${data.message || 'Unknown error'}`, 'error');
      }
    } catch (err) {
      showNotification('Error connecting to the server', 'error');
    }
  });
}

const saveEducationToDb = async (index) => {
  if (!currentProfileCode.value) {
    showNotification('Please save Personal Details first to generate an ID!', 'warning');
    return;
  }
  
  const edu = educations.value[index];
  
  if (!edu.school || !edu.degree) {
    showNotification('Please fill in School and Degree before saving.', 'warning');
    return;
  }
  
  const isNew = edu.id > 1000000;
  
  try {
    const url = isNew 
      ? `${import.meta.env.VITE_API_URL}/api/education/${currentProfileCode.value}`
      : `${import.meta.env.VITE_API_URL}/api/education/edit/${edu.id}`;
      
    const res = await fetch(url, {
      method: isNew ? 'POST' : 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        school: edu.school,
        degree: edu.degree,
        startDate: edu.startDate,
        endDate: edu.endDate,
        city: edu.city,
        description: edu.description
      })
    });
    
    const data = await res.json();
    if (!data.error && (data.id || data.success)) {
      if (isNew && data.id) {
        edu.id = data.id;
      }
      edu.isOpen = false;
      showNotification('Education saved successfully!');
    } else {
      showNotification(`Failed to save education: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification('Error connecting to save education');
  }
}

const deleteEducation = async (index) => {
  const edu = educations.value[index];
  
  if (edu.id > 1000000) {
    educations.value.splice(index, 1);
    return;
  }
  
  showConfirm('Are you sure you want to delete this education?', async () => {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/education/${currentProfileCode.value}?id=${edu.id}`, {
        method: 'DELETE'
      });
      const data = await res.json();
      if (res.ok && data.profileCode) {
        educations.value.splice(index, 1);
        showNotification('Education deleted successfully!');
      } else {
        showNotification(`Failed to delete: ${data.message || 'Unknown error'}`, 'error');
      }
    } catch (err) {
      showNotification('Error connecting to the server', 'error');
    }
  });
}

const skills = ref([
  { name: 'Java' },
  { name: 'JavaScript' },
  { name: 'Python' },
  { name: 'Git' },
  { name: 'SQL' },
  { name: 'C++' },
  { name: 'TypeScript' },
  { name: 'C#' },
  { name: 'Docker' },
  { name: 'PHP' },
  { name: 'React' },
  { name: 'MongoDB' },
  { name: 'Toad' },
  { name: 'HTML CCS 3' },
  { name: 'MS SQL server' }
])

const employments = ref([])

const addEmployment = () => {
  employments.value.push({
    id: Date.now(),
    jobTitle: '',
    employer: '',
    startDate: '',
    endDate: '',
    city: '',
    description: '',
    isOpen: true
  })
}

const toggleEmployment = (index) => {
  employments.value[index].isOpen = !employments.value[index].isOpen
}

const educations = ref([])

const addEducation = () => {
  educations.value.push({
    id: Date.now(),
    school: '',
    degree: '',
    startDate: '',
    endDate: '',
    city: '',
    description: '',
    isOpen: true
  })
}

const toggleEducation = (index) => {
  educations.value[index].isOpen = !educations.value[index].isOpen
}

const userSkills = ref([])
const skillLevels = ['Novice', 'Beginner', 'Skillful', 'Experienced', 'Expert']
const hideExperienceLevel = ref(false)

const addSkill = (skillName = '') => {
  userSkills.value.push({
    id: Date.now(),
    name: skillName,
    levelIndex: 3,
    isOpen: true
  })
}

const toggleSkill = (index) => {
  userSkills.value[index].isOpen = !userSkills.value[index].isOpen
}

const toggleSuggestedSkill = (skill) => {
  const existingIndex = userSkills.value.findIndex(s => s.name === skill.name)
  if (existingIndex > -1) {
    const us = userSkills.value[existingIndex]
    if (us.id > 1000000) {
      userSkills.value.splice(existingIndex, 1)
    } else {
      deleteSkillDb(existingIndex)
    }
  } else {
    addSkill(skill.name)
  }
}

const isSkillAdded = (skillName) => {
  return userSkills.value.some(s => s.name === skillName)
}

const saveSkillToDb = async (index) => {
  if (!currentProfileCode.value) {
    showNotification('Please save Personal Details first to generate an ID!', 'warning');
    return;
  }
  
  const us = userSkills.value[index];
  
  if (!us.name) {
    showNotification('Please fill in Skill name before saving.', 'warning');
    return;
  }
  
  const isNew = us.id > 1000000;
  
  try {
    const url = isNew 
      ? `${import.meta.env.VITE_API_URL}/api/skill/${currentProfileCode.value}`
      : `${import.meta.env.VITE_API_URL}/api/skill/edit/${us.id}`;
      
    const res = await fetch(url, {
      method: isNew ? 'POST' : 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        skill: us.name,
        level: skillLevels[us.levelIndex]
      })
    });
    
    const data = await res.json();
    if (!data.error && (data.id || data.success)) {
      if (isNew && data.id) {
        us.id = data.id;
      }
      us.isOpen = false;
      showNotification('Skill saved successfully!');
    } else {
      showNotification(`Failed to save skill: ${data.message || 'Unknown error'}`);
    }
  } catch (err) {
    showNotification('Error connecting to save skill');
  }
}

const deleteSkillDb = async (index) => {
  const us = userSkills.value[index];
  
  if (us.id > 1000000) {
    userSkills.value.splice(index, 1);
    return;
  }
  
  showConfirm('Are you sure you want to delete this skill?', async () => {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/skill/${currentProfileCode.value}?id=${us.id}`, {
        method: 'DELETE'
      });
      const data = await res.json();
      if (res.ok && data.profileCode) {
        userSkills.value.splice(index, 1);
        showNotification('Skill deleted successfully!');
      } else {
        showNotification(`Failed to delete: ${data.message || 'Unknown error'}`, 'error');
      }
    } catch (err) {
      showNotification('Error connecting to the server', 'error');
    }
  });
}

const allSkillPills = computed(() => {
  const list = [...skills.value]
  userSkills.value.forEach(us => {
    if (us.name && !list.find(s => s.name.toLowerCase() === us.name.toLowerCase())) {
      list.push({ name: us.name })
    }
  })
  return list
})
</script>

<template>
  <div class="app-container">
    <!-- Notification Toast -->
    <div v-if="notification.show" :class="['notification', notification.type]">
      {{ notification.message }}
    </div>

    <!-- Confirm Modal -->
    <div v-if="confirmModal.show" class="modal-overlay">
      <div class="modal-content">
        <h3>Confirmation</h3>
        <p>{{ confirmModal.message }}</p>
        <div class="modal-actions">
          <button class="btn-cancel" @click="cancelConfirm">Cancel</button>
          <button class="btn-confirm" @click="confirmModal.onConfirm">Delete</button>
        </div>
      </div>
    </div>

    <main class="main-content">
      <section class="section">
        <div class="load-profile-bar" style="display: flex; gap: 12px; margin-bottom: 32px; align-items: center; background: white; padding: 16px 24px; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03); border: 1px solid #e2e8f0;">
          <label style="font-size: 15px; color: var(--text-secondary); font-weight: 600;">Load existing profile:</label>
          <select v-model="loadProfileCode" class="custom-select">
            <option value="" disabled>Select Profile</option>
            <option v-for="prof in profilesList" :key="prof.id" :value="prof.id">{{ prof.firstName }} {{ prof.lastName }}</option>
          </select>
          <button class="save-btn load-btn" @click="loadProfile" :disabled="!loadProfileCode" :style="{ opacity: !loadProfileCode ? 0.5 : 1, cursor: !loadProfileCode ? 'not-allowed' : 'pointer' }">Load Data</button>
        </div>

        <h2 class="section-title">Personal Details</h2>
        
        <div class="form-grid">
          <div class="form-group">
            <label>Wanted Job Title <span class="help-icon">?</span></label>
            <input type="text" v-model="formData.jobTitle" />
          </div>
          
          <div class="form-group photo-upload">
            <div class="photo-placeholder" style="overflow: hidden;">
              <img v-if="photoDataUrl" :src="photoDataUrl" style="width: 100%; height: 100%; object-fit: cover;" />
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
            </div>
            <div style="display: flex; flex-direction: column; gap: 4px;">
              <a href="#" class="upload-link" @click="triggerPhotoUpload">{{ photoDataUrl ? 'Change photo' : 'Upload photo' }}</a>
              <a href="#" class="upload-link" v-if="photoDataUrl" @click="downloadPhoto">Download photo</a>
              <a href="#" class="upload-link" style="color: #dc3545;" v-if="photoDataUrl" @click="deletePhoto">Delete photo</a>
              <input type="file" accept="image/*" style="display: none;" ref="fileInputRef" @change="handlePhotoUpload" />
            </div>
          </div>

          <div class="form-group">
            <label>First Name</label>
            <input type="text" v-model="formData.firstName" />
          </div>
          <div class="form-group">
            <label>Last Name</label>
            <input type="text" v-model="formData.lastName" />
          </div>

          <div class="form-group">
            <label>Email</label>
            <input type="email" v-model="formData.email" />
          </div>
          <div class="form-group">
            <label>Phone</label>
            <input type="tel" v-model="formData.phone" />
          </div>

          <div class="form-group">
            <label>Country</label>
            <input type="text" v-model="formData.country" />
          </div>
          <div class="form-group">
            <label>City</label>
            <input type="text" v-model="formData.city" />
          </div>

          <div class="form-group">
            <label>Address</label>
            <input type="text" v-model="formData.address" />
          </div>
          <div class="form-group">
            <label>Postal Code</label>
            <input type="text" v-model="formData.postalCode" />
          </div>

          <div class="form-group">
            <label>Driving License <span class="help-icon">?</span></label>
            <input type="text" v-model="formData.drivingLicense" />
          </div>
          <div class="form-group">
            <label>Nationality <span class="help-icon">?</span></label>
            <input type="text" v-model="formData.nationality" />
          </div>

          <div class="form-group">
            <label>Place Of Birth</label>
            <input type="text" v-model="formData.placeOfBirth" />
          </div>
          <div class="form-group">
            <label>Date Of Birth <span class="help-icon">?</span></label>
            <input type="text" v-model="formData.dateOfBirth" />
          </div>
        </div>
        
        <div style="margin-top: 24px; text-align: right;">
          <button class="save-btn" @click="saveProfile">{{ currentProfileCode ? 'Update Profile' : 'Create Profile' }}</button>
        </div>
      </section>

      <section class="section">
        <h2 class="section-title">Professional Summary</h2>
        <p class="section-desc">Write 2-4 short & energetic sentences to interest the reader! Mention your role, experience & most importantly - your biggest achievements, best qualities and skills.</p>
        
        <RichTextEditor 
          v-model="formData.summary" 
          placeholder="e.g. Passionate science teacher with 8+ years of experience and a track record of ..." 
        />
        
        <div style="margin-top: 16px; text-align: right;">
          <button class="save-btn" @click="saveSummary">Save Summary</button>
        </div>
      </section>

      <section class="section">
        <h2 class="section-title">Employment History</h2>
        <p class="section-desc">Show your relevant experience (last 10 years). Use bullet points to note your achievements, if possible - use numbers/facts (Achieved X, measured by Y, by doing Z).</p>
        
        <div class="employment-list" v-if="employments.length > 0">
          <div v-for="(job, index) in employments" :key="job.id" class="employment-item">
            <div class="employment-header" @click="toggleEmployment(index)">
              <div class="employment-header-content">
                <div class="employment-title">{{ job.jobTitle ? (job.employer ? `${job.jobTitle} at ${job.employer}` : job.jobTitle) : (job.employer || '(Not specified)') }}</div>
                <div class="employment-date" v-if="job.startDate || job.endDate">{{ job.startDate }} {{ job.startDate && job.endDate ? '-' : '' }} {{ job.endDate }}</div>
              </div>
              <svg class="chevron-icon" :class="{ 'is-open': job.isOpen }" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            
            <div class="employment-body" v-show="job.isOpen">
              <div class="form-grid">
                <div class="form-group">
                  <label>Job title</label>
                  <input type="text" v-model="job.jobTitle" />
                </div>
                <div class="form-group">
                  <label>Employer</label>
                  <input type="text" v-model="job.employer" />
                </div>
                <div class="form-group">
                  <label>Start & End Date <span class="help-icon">?</span></label>
                  <div class="date-inputs">
                    <input type="text" placeholder="MM / YYYY" v-model="job.startDate" />
                    <input type="text" placeholder="MM / YYYY" v-model="job.endDate" />
                  </div>
                </div>
                <div class="form-group">
                  <label>City</label>
                  <input type="text" v-model="job.city" />
                </div>
              </div>
              
              <div class="form-group full-width" style="margin-top: 24px;">
                <label>Description</label>
                <RichTextEditor 
                  v-model="job.description" 
                  placeholder="e.g. Created and implemented lesson plans based on child-led interests and curiosities." 
                />
                <div class="footer-tip">
                  <span>Recruiter tip: write 200+ characters to increase interview chances</span>
                  <span>{{ job.description ? job.description.length : 0 }} / 200+</span>
                </div>
              </div>
              
              <div class="employment-actions" style="display: flex; gap: 8px;">
                <button class="save-btn" style="background-color: #fce8e8; color: #dc3545; border: 1px solid #dc3545;" @click.stop="deleteEmployment(index)">{{ job.id > 1000000 ? 'Cancel' : 'Delete' }}</button>
                <button class="save-btn" :disabled="!job.jobTitle || !job.employer" :style="{ opacity: (!job.jobTitle || !job.employer) ? 0.5 : 1, cursor: (!job.jobTitle || !job.employer) ? 'not-allowed' : 'pointer' }" @click.stop="saveEmploymentToDb(index)">Save</button>
              </div>
            </div>
          </div>
        </div>

        <button class="add-btn" @click="addEmployment"><span class="plus-icon">+</span> {{ employments.length > 0 ? 'Add one more employment' : 'Add employment' }}</button>
      </section>

      <section class="section">
        <h2 class="section-title">Education</h2>
        <p class="section-desc">A varied education on your resume sums up the value that your learnings and background will bring to job.</p>
        
        <div class="employment-list" v-if="educations.length > 0">
          <div v-for="(edu, index) in educations" :key="edu.id" class="employment-item">
            <div class="employment-header" @click="toggleEducation(index)">
              <div class="employment-header-content">
                <div class="employment-title">{{ edu.degree ? (edu.school ? `${edu.degree} at ${edu.school}` : edu.degree) : (edu.school || '(Not specified)') }}</div>
                <div class="employment-date" v-if="edu.startDate || edu.endDate">{{ edu.startDate }} {{ edu.startDate && edu.endDate ? '-' : '' }} {{ edu.endDate }}</div>
              </div>
              <svg class="chevron-icon" :class="{ 'is-open': edu.isOpen }" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            
            <div class="employment-body" v-show="edu.isOpen">
              <div class="form-grid">
                <div class="form-group">
                  <label>School</label>
                  <input type="text" v-model="edu.school" />
                </div>
                <div class="form-group">
                  <label>Degree</label>
                  <input type="text" v-model="edu.degree" />
                </div>
                <div class="form-group">
                  <label>Start & End Date <span class="help-icon">?</span></label>
                  <div class="date-inputs">
                    <input type="text" placeholder="MM / YYYY" v-model="edu.startDate" />
                    <input type="text" placeholder="MM / YYYY" v-model="edu.endDate" />
                  </div>
                </div>
                <div class="form-group">
                  <label>City</label>
                  <input type="text" v-model="edu.city" />
                </div>
              </div>
              
              <div class="form-group full-width" style="margin-top: 24px;">
                <label>Description</label>
                <RichTextEditor 
                  v-model="edu.description" 
                  placeholder="e.g. Graduated with honors, participated in coding club." 
                />
              </div>
              
              <div class="employment-actions" style="display: flex; gap: 8px;">
                <button class="save-btn" style="background-color: #fce8e8; color: #dc3545; border: 1px solid #dc3545;" @click.stop="deleteEducation(index)">{{ edu.id > 1000000 ? 'Cancel' : 'Delete' }}</button>
                <button class="save-btn" :disabled="!edu.school || !edu.degree" :style="{ opacity: (!edu.school || !edu.degree) ? 0.5 : 1, cursor: (!edu.school || !edu.degree) ? 'not-allowed' : 'pointer' }" @click.stop="saveEducationToDb(index)">Save</button>
              </div>
            </div>
          </div>
        </div>

        <button class="add-btn" @click="addEducation"><span class="plus-icon">+</span> {{ educations.length > 0 ? 'Add one more education' : 'Add education' }}</button>
      </section>

      <section class="section">
        <h2 class="section-title">Skills</h2>
        <p class="section-desc">Choose 5 of the most important skills to show your talents! Make sure they match the keywords of the job listing if applying via an online system.</p>
        
        <div class="toggle-group">
          <label class="toggle-switch">
            <input type="checkbox" v-model="hideExperienceLevel" />
            <span class="toggle-slider"></span>
          </label>
          <span class="toggle-label" @click="hideExperienceLevel = !hideExperienceLevel">Don't show experience level</span>
        </div>

        <div class="skills-list">
          <button 
            v-for="skill in allSkillPills" 
            :key="skill.name"
            class="skill-pill"
            :class="{ 'is-selected': isSkillAdded(skill.name) }"
            @click="toggleSuggestedSkill(skill)"
          >
            {{ skill.name }} <span>{{ isSkillAdded(skill.name) ? '✔' : '+' }}</span>
          </button>
        </div>

        <div class="employment-list" v-if="userSkills.length > 0">
          <div v-for="(us, index) in userSkills" :key="us.id" class="employment-item">
            <div class="employment-header" @click="toggleSkill(index)">
              <div class="employment-header-content">
                <div class="employment-title">{{ us.name || '(Not specified)' }}</div>
                <div class="employment-date" v-if="!hideExperienceLevel">{{ skillLevels[us.levelIndex] }}</div>
              </div>
              <svg class="chevron-icon" :class="{ 'is-open': us.isOpen }" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            
            <div class="employment-body" v-show="us.isOpen">
              <div class="form-grid">
                <div class="form-group">
                  <label>Skill</label>
                  <input type="text" v-model="us.name" />
                </div>
                
                <div class="form-group" v-if="!hideExperienceLevel">
                  <label>Level — <span class="level-label" :class="'text-level-' + us.levelIndex">{{ skillLevels[us.levelIndex] }}</span></label>
                  <div class="level-bars" :class="'level-' + us.levelIndex">
                    <div 
                      v-for="n in 5" 
                      :key="n" 
                      class="level-bar"
                      :class="{ 
                        'is-active': n - 1 < us.levelIndex,
                        'is-current': n - 1 === us.levelIndex 
                      }"
                      @click="us.levelIndex = n - 1"
                    ></div>
                  </div>
                </div>
              </div>
              
              <div class="employment-actions" style="display: flex; gap: 8px;">
                <button class="save-btn" style="background-color: #fce8e8; color: #dc3545; border: 1px solid #dc3545;" @click.stop="deleteSkillDb(index)">{{ us.id > 1000000 ? 'Cancel' : 'Delete' }}</button>
                <button class="save-btn" :disabled="!us.name" :style="{ opacity: !us.name ? 0.5 : 1, cursor: !us.name ? 'not-allowed' : 'pointer' }" @click.stop="saveSkillToDb(index)">Save</button>
              </div>
            </div>
          </div>
        </div>

        <button class="add-btn" @click="addSkill()"><span class="plus-icon">+</span> {{ userSkills.length > 0 ? 'Add one more skill' : 'Add skill' }}</button>
      </section>
    </main>
  </div>
</template>

<style scoped>
.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 16px 24px;
  border-radius: 8px;
  color: white;
  font-weight: 500;
  z-index: 1000;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  animation: slideIn 0.3s ease-out;
}
.notification.success { background-color: #10b981; }
.notification.error { background-color: #ef4444; }
.notification.warning { background-color: #f59e0b; color: #fff; }

@keyframes slideIn {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001;
}

.modal-content {
  background: white;
  padding: 24px;
  border-radius: 8px;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1);
}

.modal-content h3 {
  margin-top: 0;
  margin-bottom: 12px;
  color: #1a1a1a;
}

.modal-content p {
  color: #4a5568;
  margin-bottom: 24px;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.btn-cancel {
  padding: 8px 16px;
  background: #e2e8f0;
  border: none;
  border-radius: 4px;
  color: #4a5568;
  cursor: pointer;
  font-weight: 500;
}

.btn-cancel:hover { background: #cbd5e1; }

.btn-confirm {
  padding: 8px 16px;
  background: #ef4444;
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
  font-weight: 500;
}

.btn-confirm:hover { background: #dc2626; }

.app-container {
  width: 100%;
  min-height: 100vh;
  background-color: var(--surface-color);
  display: flex;
  flex-direction: column;
}

.custom-select {
  appearance: none;
  -webkit-appearance: none;
  background-color: #f1f5f9;
  border: 2px solid transparent;
  border-radius: 8px;
  padding: 10px 40px 10px 16px;
  font-size: 15px;
  font-weight: 500;
  color: #1e293b;
  cursor: pointer;
  outline: none;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  min-width: 240px;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%2364748b' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 14px center;
  background-size: 18px;
}

.custom-select:hover {
  background-color: #e2e8f0;
}

.custom-select:focus {
  background-color: #ffffff;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 4px rgba(26, 115, 232, 0.15);
}

.load-btn {
  padding: 10px 20px;
  margin: 0;
  border-radius: 8px;
  font-weight: 600;
  letter-spacing: 0.3px;
  box-shadow: 0 2px 4px rgba(26, 115, 232, 0.2);
}

.load-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(26, 115, 232, 0.3);
}

.load-btn:active:not(:disabled) {
  transform: translateY(0);
}

.header {
  text-align: center;
  padding: 32px 0 0;
  position: relative;
  background: linear-gradient(180deg, #d3d5db 0%, #e8e9ec 100%);
  height: 100px;
}

.header-top {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: #fff;
  font-size: 16px;
  font-weight: 500;
  position: absolute;
  top: 16px;
  left: 50%;
  transform: translateX(-50%);
  cursor: pointer;
}

.icon-down {
  opacity: 0.8;
}

.header-step {
  margin: 0;
  font-size: 36px;
  font-weight: 800;
  color: #000;
  background: #fff;
  position: absolute;
  bottom: -24px;
  left: 50%;
  transform: translateX(-50%);
  padding: 0 16px;
}

.main-content {
  max-width: 820px;
  margin: 0 auto;
  padding: 60px 24px 100px;
  width: 100%;
}

.section {
  margin-bottom: 56px;
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.section-title {
  font-size: 22px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 8px;
}

.section-desc {
  font-size: 14px;
  color: var(--text-secondary);
  margin-bottom: 24px;
  line-height: 1.6;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px 32px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-size: 14px;
  color: var(--text-secondary);
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
}

.help-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 1px solid var(--primary-color);
  font-size: 11px;
  color: var(--primary-color);
  cursor: pointer;
  transition: all 0.2s;
  font-weight: 600;
}

.help-icon:hover {
  background-color: var(--primary-color);
  color: white;
}

input[type="text"],
input[type="email"],
input[type="tel"],
select {
  width: 100%;
  padding: 14px 16px;
  background-color: var(--input-bg);
  border: 2px solid transparent;
  border-radius: var(--radius-md);
  font-size: 15px;
  color: var(--text-primary);
  transition: all 0.2s ease;
  font-family: inherit;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  background-color: #f7f9fa;
  border-bottom: 2px solid var(--primary-color);
}

.photo-upload {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 16px;
  margin-top: 8px;
}

.photo-placeholder {
  width: 60px;
  height: 60px;
  background-color: var(--input-bg);
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #a0a4ab;
}

.upload-link {
  color: var(--primary-color);
  text-decoration: none;
  font-size: 15px;
  font-weight: 500;
  transition: color 0.2s;
}

.upload-link:hover {
  color: var(--primary-hover);
  text-decoration: underline;
}

/* Rich Text Editor */
.rich-text-editor {
  background-color: var(--input-bg);
  border-radius: var(--radius-md);
  overflow: hidden;
  border: 2px solid transparent;
  transition: background-color 0.2s, border-color 0.2s;
}

.rich-text-editor:focus-within {
  background-color: #f7f9fa;
  border-bottom: 2px solid var(--primary-color);
}

.rte-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background-color: transparent;
}

.rte-tools {
  display: flex;
  align-items: center;
  gap: 4px;
}

.tool-btn {
  background: none;
  border: none;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  cursor: pointer;
  color: #4a5568;
  font-family: serif;
  font-size: 16px;
  transition: all 0.2s;
}

.tool-btn:hover {
  background-color: rgba(0,0,0,0.05);
  color: #000;
}

.font-bold { font-weight: bold; }
.font-italic { font-style: italic; }
.font-underline { text-decoration: underline; }
.font-strike { text-decoration: line-through; }

.divider {
  width: 1px;
  height: 16px;
  background-color: #cbd5e1;
  margin: 0 8px;
}

.pre-written-btn {
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: color 0.2s;
}

.pre-written-btn:hover {
  color: var(--primary-color);
}

.plus-circle {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  background-color: var(--primary-color);
  color: white;
  border-radius: 50%;
  font-size: 14px;
  font-weight: bold;
  line-height: 1;
}

.rich-text-editor :deep(.tiptap) {
  width: 100%;
  min-height: 150px;
  padding: 0 16px 16px;
  border: none;
  background: transparent;
  font-family: inherit;
  font-size: 15px;
  color: var(--text-primary);
  line-height: 1.6;
  outline: none;
}

.rich-text-editor :deep(.tiptap p.is-editor-empty:first-child::before) {
  color: #a0aab8;
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}

.tool-btn.is-active {
  background-color: #e2e8f0;
  color: #000;
}

/* Employment History */
.employment-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 24px;
}

.employment-item {
  border: 1px solid #e2e8f0;
  border-radius: var(--radius-md, 8px);
  background-color: #fff;
  overflow: hidden;
}

.employment-header {
  padding: 16px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  transition: background-color 0.2s;
}

.employment-header:hover {
  background-color: #f8fafc;
}

.employment-title {
  font-weight: 600;
  color: var(--text-primary);
  font-size: 16px;
  margin-bottom: 4px;
}

.employment-date {
  font-size: 14px;
  color: var(--text-secondary);
}

.chevron-icon {
  color: #a0aec0;
  transition: transform 0.2s;
}

.chevron-icon.is-open {
  transform: rotate(180deg);
}

.employment-body {
  padding: 0 24px 24px;
}

.date-inputs {
  display: flex;
  gap: 16px;
}

.date-inputs input {
  flex: 1;
}

.full-width {
  grid-column: 1 / -1;
}

.employment-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

.save-btn {
  background-color: var(--primary-color, #1a73e8);
  color: white;
  border: none;
  border-radius: var(--radius-md, 8px);
  padding: 10px 24px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.save-btn:hover {
  background-color: var(--primary-hover, #1557b0);
}

.footer-tip {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #a0aec0;
  margin-top: 8px;
}

.add-btn {
  background: none;
  border: none;
  color: var(--primary-color);
  font-size: 15px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 8px 0;
  transition: color 0.2s;
}

.add-btn:hover {
  color: var(--primary-hover);
}

.plus-icon {
  font-size: 20px;
  font-weight: 400;
  line-height: 1;
}

.skills-list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 24px;
}

.skill-pill {
  background-color: var(--input-bg);
  border: none;
  padding: 8px 16px;
  border-radius: var(--radius-md);
  font-size: 14px;
  color: var(--text-secondary);
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.2s;
}

.skill-pill span {
  color: #a0aab8;
  font-size: 16px;
  line-height: 1;
  font-weight: 400;
}

.skill-pill:hover {
  background-color: #e2e8f0;
  color: var(--text-primary);
}

.skill-pill.is-selected {
  background-color: var(--primary-color, #1a73e8);
  color: white;
}

.skill-pill.is-selected span {
  color: white;
}

.level-bars {
  display: flex;
  height: 46px;
  background-color: #edf2f7;
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
}

.level-bar {
  flex: 1;
  background-color: transparent;
  cursor: pointer;
  transition: background-color 0.2s;
  border-right: 1px solid rgba(0,0,0,0.05);
}

.level-bar:last-child {
  border-right: none;
}

/* Dynamic level colors */
.level-bars.level-0 .level-bar.is-current { background-color: #ef4444; }
.level-bars.level-0 .level-bar.is-active { background-color: #fee2e2; }
.text-level-0 { color: #ef4444; }

.level-bars.level-1 .level-bar.is-current { background-color: #f97316; }
.level-bars.level-1 .level-bar.is-active { background-color: #ffedd5; }
.text-level-1 { color: #f97316; }

.level-bars.level-2 .level-bar.is-current { background-color: #f59e0b; }
.level-bars.level-2 .level-bar.is-active { background-color: #fef3c7; }
.text-level-2 { color: #f59e0b; }

.level-bars.level-3 .level-bar.is-current { background-color: #48bb78; }
.level-bars.level-3 .level-bar.is-active { background-color: #e6f6ec; }
.text-level-3 { color: #48bb78; }

.level-bars.level-4 .level-bar.is-current { background-color: #8b8df8; }
.level-bars.level-4 .level-bar.is-active { background-color: #e0e7ff; }
.text-level-4 { color: #8b8df8; }

.level-label {
  font-weight: 600;
}

.toggle-group {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: 44px;
  height: 24px;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #cbd5e1;
  transition: .4s;
  border-radius: 24px;
}

.toggle-slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .toggle-slider {
  background-color: var(--primary-color, #1a73e8);
}

input:checked + .toggle-slider:before {
  transform: translateX(20px);
}

.toggle-label {
  font-size: 15px;
  color: var(--text-primary);
  cursor: pointer;
  user-select: none;
}

@media (max-width: 640px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>

