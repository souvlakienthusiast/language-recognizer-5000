**Documentation**
for the “language recognizer 5000” program

**User Manual**

The languages this program is able to recognize are Czech, English, French, German, Polish, Dutch, Hungarian, Lithuanian, Italian & Spanish.
The steps the user must take to use this program are quite straightforward.
Preparation steps:

	1.	Download the folder ‘language detection program’ in which the program is exported in. It will contain the python file ‘language recognizer 5000.py’, which contains the code itself. It will also contain another folder titled JSON_language_profiles, where the necessary data for text recognition is located. Make sure the JSON_language_profiles folder is in the same directory as the python script and do not change the names of any files.

	2.	Download all necessary python packages for the code to run from the requirements.txt text file.

	3.	Open ‘language recognizer 5000.py’ in your preferred interpreter.

Using the program

	1.	Run the program.
	
	2.	You will be prompted to enter the text whose language you want to detect. This text should be no shorter than 30 letters, where non-letter characters such as numbers, punctuation marks or whitespaces are not counted.
	
	3.	One of the three following possibilities will take place:
		•	The program has detected the language the text has been written in. In this case it will return the input text, along with the name of the detected language and a dictionary of the scores that were attributed to each language (for further information about the scoring system see the more detailed Implementation & Script Description section).
		
		•	The language was not detected. This simply means that the program was not able to tell with certainty what language the input text is written in. It can occur either when the input text is written in one of the languages not included in our 10 chosen ones, or simply if the text is too short for the program to recognize, despite being over 30 characters long. The input text and scores are again returned to be available to be reviewed by the user.
		
		•	Your input text was shorter than 30 characters. An error message and the input text are returned for review.

**Principle of the Program**

The program detects languages based on the n-gram language model. An n-gram is a sequence of n letters, where n is a natural number. First, n-grams are extracted out of a text in the following manner. Let’s have a sample text ‘apples’, the tri-grams (i.e., n-grams for n=3) to be extracted from this text are ‘app’, ‘ppl’, ‘ple’ & ‘les’. 
For each language, you must start with a dataset of text from which to extract the n-grams, or in the case of my program, tri-grams. Each language gets a profile, where the tri-grams will be saved. Once the tri-grams have been extracted, you calculate the probability of each one of them appearing in the dataset you used, meaning the frequency of the occurrence of the specific tri-gram divided by the total number of tri-grams extracted.
Now you must extract tri-grams from the input text and compare them to the tri-grams of each language.
This is implemented by giving each language a score given by this comparison, roughly speaking the more similar the tri-grams of the input text are to the tri-grams of a certain language, the higher score this language gets. The detected language is the one with the highest score.

**Implementation & Script Description**

At the beginning of the script, there are several functions that are not included under a class. This is simply because many of them are used throughout the program in methods within classes and it would be impractical for them to be part of a class, considering readability is not much of a concern in a code of this length.
Many of these functions utilize the docx module, they were used primarily to parse the dataset of texts that were saved in docx files into ‘clean text’ without formatting, diacritics, whitespaces, non-letter characters etc., to make the extraction of tri-grams simpler. These functions were at some point called to rewrite the docx files in this preferred format, even though they are not actually used in the final program. I decided to keep them in the code anyway as this is a school project and they show how I handled the original texts.
I will now describe what purpose each class in my code serves and the methods used to achieve these goals.

***DatasetBuilder Class***

This class was used solely to convert the text data from the docx files into a python dictionary to be later used in other classes, most importantly to build language profiles that work with this data. The ‘create_dataset’ method returned dictionaries in the following format:
{
CZ: {'name': 'Czech', 'texts': [ 'hody hody doprovody', 'dejte vejce malovany' ]}
FR: {'name': 'French', 'texts': [ 'ta grandmere en string', 'me met l eau a la bouche' ]}
}
Later, the ‘texts’ would be replaced with their tri-grams and the trigrams’ probabilities and saved into JSON files, more on that later.

***LanguageDetector Class***

This class is the core of the program, it is where the language detection algorithm is located.
Here we have methods to add and build language profiles from the dataset built using the *DatasetBuilder* class. Tri-grams are extracted from the ‘texts’ using the *Counter* data structure from the collections module, this data structure is very similar to a classic dictionary, but it is more practical for counting the occurrences of n-grams, as it is specifically designed for counting the frequency of elements in an iterable and is slightly more efficient at this task.
The language profiles have a structure very similar to that of the dictionary returned by the ‘create_dataset’ method from the *DatasetBuilder* class, excepts the ‘texts’ are replaced with ‘n-grams’, and each n-gram is paired with its probability of occurrence in the ‘texts’, i.e. the frequency of the given tri-gram divided by the total number of tri-grams extracted from ‘texts’.
When detecting the language of the input text, each language is attributed a score by the ‘calculate_score’ method. A language has an initial score of 0. The method iterates over all of the tri-grams of the input text and checks if the given tri-gram is also part of the given languages profile. If it is, the frequency of the tri-gram in the input text is multiplied by the probability of the occurrence of the tri-gram in the language profile and this number is then added onto the score.
The ’detect_language’ method is where it all comes together. It extracts tri-grams from the input text, calculates the scores of all the languages and finds the one with the highest score, which it returns as the detected language, if it fits within certain thresholds. The input text may not be shorter than 30 characters, as the program provides very inconsistent results with such short texts. The texts are further divided into short texts (30 to 50 characters), medium texts (50 to 200 characters) and long texts (200+ characters). The minimum score thresholds for the language to be considered detected are 0.015, 0.03 and 0.13 respectively. These figures come from trial and error while I was testing the functionality of the program. I have yet to find a better way to determine these thresholds.
The ’detect_language’ method also provides the user with the dictionary of all the language scores.
The ‘get_results’ method calls the ‘detect_language’ method and prints the results of the detection in a concise, well-readable form. It prints the input text, the detected language’s name, as well as the aforementioned dictionary of language scores.

***JSONLanguageProfiles Class***

Since there are too many tri-grams in each language profile to store within the code itself, I decided to store each profile in a separate JSON, or JavaScript Object Notation, file. I used this rather than a format like docx, because it is much easier to work with within a python script (very easily converted to a python dictionary) and it also saves space compared to docx files, which have many more functions that would be completely useless for the simple task of storing profiles.
This is where the language profiles are loaded from while detecting the language. The code here is heavily dependent on files being stored under the right names and directories because they are found under specific paths, hence why I mentioned not to change the file names and to have the python file in the correct folder in the User Manual section. 

***main()***

In the ‘main‘ function (the one that actually gets run when you run the program), we have a dictionary containing the language codes and names of our 10 languages. From this dictionary, the language profiles are built within the instance of our *LanguageDetector* class using the data from our JSON files. After that, we simply take the input text from the user and use the ‘get_results’ method of the *LanguageDetector* class to obtain the desired output.







