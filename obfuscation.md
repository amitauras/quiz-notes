1. Store questions in database with an `answered` field to track if user has answered it.

2. Have an `/api/quiz` endpoint to start quiz, initialize user session data.

3. Have a `/api/question` endpoint that returns next unanswered question.

   - Check if user session exists and valid
   - Find first question where `answered = false`  
   - Return that question
   - Questions can have options and correct answer

4. Have `/api/answer` endpoint  

   - Accept question ID and user answer 
   - Verify if correct answer
   - Mark question as `answered = true` in DB
   - Return score or messages like correct/wrong

5. To get next question, client calls `/api/question` again.

   - Your logic only returns unanswered questions
   - User can't access all questions at once

6. To end quiz 
   - Have endpoint like `/api/end`  
   - Or after last question return some final message


```js
// Question model
const Question = mongoose.model('Question', {
  text: String,
  options: [String], 
  answer: String,
  answered: {type: Boolean, default: false}
})

// Start quiz
app.post('/api/quiz/start', (req, res) => {
  
  // Initialize session  
  req.session.questions = []; 
  req.session.score = 0;

  res.send("Quiz started!")
})

// Get next question
app.get('/api/question', (req, res) => {
  
  // Check session exists
  if(!req.session.questions) {
    return res.status(400).send("Session expired") 
  }

  // Find next unanswered question
  Question.findOne({answered: false}, (err, question) => {
    if(err || !question) {
      return res.send("Quiz ended!"); 
    }
    
    // Add to user session  
    req.session.questions.push(question._id);

    // Return question
    res.send(question);

  })

})


// Submit user's answer
app.post('/api/answer', (req, res) => {

  const {questionId, answer} = req.body;

  // Validate 
  if(!questionId || !answer) {
    return res.status(400).send("Invalid request!");
  }

  // Find question object  
  Question.findById(questionId, (err, question) => {
    
    if(err || !question) {
      return res.status(400).send("Question not found!");
    }

    // Evaluate answer 
    if(question.answer === req.body.answer) {
      req.session.score += 10; // Award points   
      question.answered = true;  // Mark as answered
    }  

    question.save();

    res.send("Answer evaluated!");
  
  });

})
```
