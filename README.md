# Online-cpct-moke-tests
import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

const questions = [
  { question: "CPU का पूरा नाम क्या है?", options: ["Central Processing Unit", "Computer Processing Unit", "Central Processor Unit", "Core Processing Unit"], answer: "Central Processing Unit" },
  { question: "RAM किस प्रकार की मेमोरी होती है?", options: ["Permanent", "Volatile", "Non-Volatile", "Cache"], answer: "Volatile" }
];

export default function MockTestHome() {
  const [userName, setUserName] = useState("");
  const [testStarted, setTestStarted] = useState(false);
  const [currentQuestion, setCurrentQuestion] = useState(0);
  const [selectedOption, setSelectedOption] = useState(null);
  const [score, setScore] = useState(0);
  const [testCompleted, setTestCompleted] = useState(false);

  const startTest = () => {
    if (userName.trim() !== "") {
      setTestStarted(true);
    }
  };

  const handleAnswer = (option) => {
    setSelectedOption(option);
    if (option === questions[currentQuestion].answer) {
      setScore(score + 1);
    }
  };

  const nextQuestion = () => {
    if (currentQuestion < questions.length - 1) {
      setCurrentQuestion(currentQuestion + 1);
      setSelectedOption(null);
    } else {
      setTestCompleted(true);
    }
  };

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
      <Card className="w-full max-w-md p-6 text-center shadow-lg rounded-2xl">
        <CardContent>
          <h1 className="text-2xl font-bold mb-4">CPCT Mock Test</h1>
          {!testStarted ? (
            <div>
              <Input
                type="text"
                placeholder="Enter your name"
                value={userName}
                onChange={(e) => setUserName(e.target.value)}
                className="mb-4"
              />
              <Button onClick={startTest} className="w-full">Start Test</Button>
            </div>
          ) : testCompleted ? (
            <div>
              <h2 className="text-xl font-semibold mb-4">Test Completed!</h2>
              <p>Your Score: {score} / {questions.length}</p>
            </div>
          ) : (
            <div>
              <h2 className="text-xl font-semibold mb-4">{questions[currentQuestion].question}</h2>
              {questions[currentQuestion].options.map((option, index) => (
                <Button 
                  key={index} 
                  onClick={() => handleAnswer(option)} 
                  className={`w-full mb-2 ${selectedOption === option ? "bg-blue-500 text-white" : ""}`}>
                  {option}
                </Button>
              ))}
              <Button onClick={nextQuestion} className="w-full mt-4" disabled={!selectedOption}>
                Next
              </Button>
            </div>
          )}
        </CardContent>
      </Card>
    </div>
  );
}
