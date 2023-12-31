<template>
  <nav>
    <ul class="nav-links">
      <li>
        <RouterLink to="/student/schedule">View Schedule</RouterLink>
      </li>
      <li>
        <RouterLink to="/student/enroll">Search and Enroll</RouterLink>
      </li>
      <li>
        <LogOut />
      </li>
    </ul>
  </nav>
  <main class="enroll-view">
    <popup v-if="popupTriggers.buttonTrigger" :TogglePopup="() => TogglePopup('buttonTrigger')">
      <h2>{{ alertMsg }}</h2>
    </popup>
    <h2>Search and Enroll in Courses</h2>
	<h3>
		(If you do not see any courses populate after pressing 'Search', please wait a few seconds then type another character in the search bar.)
	</h3>
    <div class="search">

      <input type="text" id="searchBar" v-model="course_search_bar" placeholder="Search for class name or title (case sensitive)" />
      <input type="text" id="searchBar2" v-model="studentID_input" placeholder="Enter your studentID to enroll in a course" />
      <button class ="sButton" type="button" @click="getCourses" @keydown.enter="getCourses">Search</button>

      <!-- this somehow updates the courses, not sure how -->
      <courseEntry v-if="popupTriggers.courseAvailable" v-for="course in courseCatalog" :key="course">
      </courseEntry>

      <div class="course-list" v-for="course in courseCatalog" :key="course">
        <h4>{{ course[0]+' - '+course[2]}}</h4>
		    <h5>{{ 'Professor: '+course[7]+', '+course[1]+' credit(s)' }}</h5>
		    <h5>{{ course[3]+'-'+course[4]+' - '+course[5] }}</h5>
		    <h5>{{ 'Current enrollment: '+course[9]+'/'+course[8]+' ('+(course[8]-course[9])+' open seats)'}}</h5>
        <h5>{{ }}</h5>
		<button class ="eButton" type="button" @click="enrollStudentInCourse(course[0])">Enroll in {{course[0]}}</button>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import { searchMultipleCourseCatalog, enrollStudent } from "../util/api-setup";
import LogOut from "@/components/buttons/logout-button.vue";
import popup from "@/components/course-info-popup.vue";
import courseEntry from "@/components/course-snippet.vue";

var alertMsg = ref("");
const course_search_bar = ref("");
var courseCatalog = [];


// testing student id -- CHANGE LATER
// const studentID = "abc12345"
const studentID_input = ref("");
// var studentID = studentID_input.value;

/* this should get the number of credits a student has from dynamo
(if we configure the enrollment button to display conditionally rather than how we have it now) */
const MAX_CREDITS = 19



//get courses from dynamoDB courseCatalog
const getCourses = async () => {
  const { data, error } = await searchMultipleCourseCatalog(course_search_bar.value);

  if (data) {
    courseCatalog = data.courses;
    console.log(data.courses);
    TogglePopup('courseAvailable');
  }

  if (error) {
    courseCatalog = ["There was an error, please search again."];
  }
};

//enroll student in a course
const enrollStudentInCourse = async (course) => {
  console.log("Attempting to enroll in " + course)
  const studentID = studentID_input.value;
  const {data, error} = await enrollStudent({studentID, course});

  if (data) {
    // Possible Outputs:
	// "Enrollment SUCCESSFUL - You have been enrolled!!"
	// "You are already enrolled in this course." (course in student[enrolled_courses])
	// "Enrollment FAILED - You are enrolled in too many credits to take this course." (enrolled_credits + course_credits >= max_credits)
	// "Enrollment FAILED - This course is at maximum capacity, you cannot enroll at this time." (current_enrollment == max_enrollment)
	// console.log(data.body);
	
	// alternate way of logging
    // console.log(data.body);
    if (data.body.includes("have been enrolled")) {
      // console.log("You have been enrolled!!");
      alertMsg = "You have been enrolled in " + course;
    } else if (data.body.includes("maximum capacity")){
      // console.log("Course is at max capacity");
      alertMsg = course + " is at max capacity.";
    } else if (data.body.includes("too many credits")) {
      // console.log("You are enrolled in too many credits");
      alertMsg = "You are enrolled in too many credits.";
    } else if (data.body.includes("already enrolled")) {
      // console.log(data.body);
      alertMsg = "You are already enrolled in " + course;
    } else if (data.body.includes("Student not found")) {
      alertMsg = "Student not found, check your student ID input!"
    } else {
      // console.log("There was an error, please try again.");
      alertMsg = "There was an error, please try again.";
    }
    TogglePopup('buttonTrigger');
  } else {
    // console.log(error);
    alertMsg = "There was an error, please try again.";
    TogglePopup('buttonTrigger');
  }

};

const popupTriggers = ref({
  buttonTrigger: false,
  courseAvailable: false
});

const TogglePopup = (trigger) => {
  popupTriggers.value[trigger] = !popupTriggers.value[trigger]
};

</script>
<style>
  .home {
    padding: 1rem;
  }
  
  /* page heading */
  .enroll-view h2 {
    font-size: 1.75rem;
    margin-bottom: .75rem;
	text-align: center;
	font-weight: bold;
  }
  
  /* page sub-heading */
  .enroll-view h3 {
    font-size: 1rem;
    margin-bottom: 1.5rem;
	text-align: center;
  }
  
  /* course title */
  .enroll-view h4 {
	font-weight: bold;
	text-align: center;
	/* add any extra params to on-screen text here*/
  }
  
  /* course descriptors */
  .enroll-view h5 {
	text-align: center;
	/* add any extra params to on-screen text here*/
  }
  
	/* hyperlinks (i.e. enroll button) */
  .enroll-view a {
	display: flex;
	justify-content: center;
    margin-bottom: 1.5rem;
	/* add any extra params to on-screen text here*/
  }

	/* centers the search button */
  .search {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

	/* search button */
  .sButton {
    height: 2rem;
    width: 8rem;
    background-color: #1212c2;
    -webkit-text-fill-color: #fef1fc;
    margin: 0rem 0rem 1rem 0rem;
  }

	/* centers the enrollment buttons */
  .course-list {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
	/* enrollment button */
	/* can probably delete the following code: */
	/* display: flex;
	justify-content: center;
    align-items: center;
	text-align: center; */
	
  .eButton {
    background-color: #00CC00;		/* green */
    -webkit-text-fill-color: #FFFFFF;	/* white */
    height: 2rem;
    width: 11rem;
    margin: .5rem 0rem 1rem 0rem;
  }
  
	/* enrollment failed button */
  .efButton {
    background-color: #CCCCCC;	/* light grey */
    -webkit-text-fill-color: #000000;	/* black */
    height: 2rem;
    width: 11rem;
    margin: .5rem 0rem 1rem 0rem;
  }

	/* search bar */
  .enroll-view input {
    display: block;
    width: 500px;
    margin: 20px auto;
    padding: 10px 45px;
  /* background: white url("assets/searchIcon.svg") no-repeat 15px center; */
    background-size: 15px 15px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px,
     rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    }
</style>