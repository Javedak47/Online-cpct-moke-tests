import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Document, Page, pdfjs } from "react-pdf";

pdfjs.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjs.version}/pdf.worker.js`;

export default function MockTestHome() {
  const [userName, setUserName] = useState("");
  const [testStarted, setTestStarted] = useState(false);
  const [pdfFile, setPdfFile] = useState(null);
  const [numPages, setNumPages] = useState(null);
  const [questions, setQuestions] = useState([]);
  const [currentQuestion, setCurrentQuestion] = useState(0);
  const [selectedOption, setSelectedOption] = useState(null);
  const [score, setScore] = useState(0);
  const [testCompleted, setTestCompleted] = useState(false);

  const startTest = () => {
    if (userName.trim() !== "" && questions.length > 0) {
      setTestStarted(true);
    }
  };

  const handlePdfUpload = (event) => {
    const file = event.target.files[0];
    setPdfFile(file);
  };

  const extractTextFromPdf = async () => {
    if (!pdfFile) return;
    
    const reader = new FileReader();
    reader.readAsArrayBuffer(pdfFile);
    reader.onload = async () => {
      const loadingTask = pdfjs.getDocument({ data: reader.result });
      const pdf = await loadingTask.promise;
      let extractedQuestions = [];
      
      for (let i = 1; i <= pdf.numPages; i++) {
        const page = await pdf.getPage(i);
        const textContent = await page.getTextContent();
        const textItems = textContent.items.map(item => item.str);
        extractedQuestions.push(...parseQuestions(textItems));
      }
      
      setQuestions(extractedQuestions);
    };
  };

  const parseQuestions = (textItems) => {
    let parsedQuestions = [];
    let questionRegex = /(\d+\.\s+.*?)(?:\n|$)/g;
    let optionRegex = /\(([A-D])\)\s+(.*)/g;
    
    let matches;
    let question = null;
    let options = [];
    let answer = "";
    
    textItems.forEach(line => {
      if ((matches = questionRegex.exec(line)) !== null) {
        if (question) {
          parsedQuestions.push({ question, options, answer });
        }
        question = matches[1];
        options = [];
      } else if ((matches = optionRegex.exec(line)) !== null) {
        options.push(matches[2]);
        if (line.includes("*")) {
          answer = matches[2];
        }
      }
    });
    
    if (question) {
      parsedQuestions.push({ question, options, answer });
    }
    
    return parsedQuestions;
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
              <input type="file" accept="application/pdf" onChange={handlePdfUpload} className="mb-4" />
              {pdfFile && <p className="mb-2 text-green-600">Uploaded: {pdfFile.name}</p>}
              <Button onClick={extractTextFromPdf} className="w-full mb-2">Extract Questions</Button>
              <Button onClick={startTest} className="w-full" disabled={questions.length === 0}>Start Test</Button>
            </div>
          ) : testCompleted ? (
            <div>
              <h2 className="text-xl font-semibold mb-4">Test Completed!</h2>
              <p>Your Score: {score} / {questions.length}</p>
            </div>
          ) : (
            <div>
              <h2 className="text-xl font-semibold mb-4">{questions[currentQuestion]?.question}</h2>
              {questions[currentQuestion]?.options.map((option, index) => (
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
