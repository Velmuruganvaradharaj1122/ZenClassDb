#  MongoDB Zen Class Dataset
---

##  Collections Overview

- `users`: Student details and CodeKata scores
- `codekata`: Problems solved by each student
- `attendance`: Attendance records
- `topics`: Topics covered with dates
- `tasks`: Tasks assigned with dates
- `company_drives`: Placement drives and student participation
- `mentors`: Mentor details and mentee mapping
- `task_submissions`: Task submission status by students

---

##  Sample Data Insertion

Run these commands in the MongoDB shell (`mongosh`) after switching to your database:

```js

use zenClassDB


1️)1Topics and Tasks in October 2020
js
db.topics.find({ date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } })
db.tasks.find({ date: { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") } })
2️) Company Drives Between 15–31 Oct 2020
js
db.company_drives.find({ drive_date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") } })
3️) Company Drives and Students Who Appeared
js
db.company_drives.find({}, { company: 1, drive_date: 1, students_appeared: 1 })
4️) CodeKata Scores by User
js
db.codekata.find({}, { user_name: 1, problems_solved: 1 })
5️) Mentors with More Than 15 Mentees
js
db.mentors.find({ $expr: { $gt: [{ $size: "$mentees" }, 15] } })
6️) Users Absent and Didn’t Submit Tasks (15–31 Oct)
js
db.attendance.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") },
      status: "absent"
    }
  },
  {
    $lookup: {
      from: "task_submissions",
      let: { uname: "$user_name", adate: "$date" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$user_name", "$$uname"] },
                { $eq: ["$submitted", false] },
                { $gte: ["$date", ISODate("2020-10-15")] },
                { $lte: ["$date", ISODate("2020-10-31")] }
              ]
            }
          }
        }
      ],
      as: "unsubmitted_tasks"
    }
  },
  {
    $match: {
      "unsubmitted_tasks.0": { $exists: true }
    }
  },
  {
    $count: "absent_and_not_submitted"
  }
])

