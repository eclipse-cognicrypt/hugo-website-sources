This tutorial page describes how one would use CogniCrypt's code-generation feature. For a more technical description of how the code generation works in the background, please refer to [Tool Paper on CogniCrypt](http://bodden.de/pubs/knr+17cognicrypt.pdf) published at ASE 2017.

# Launching the Code Generator

CogniCrypt's code generator can most easily be triggered by clicking the CogniCrypt button in the Eclipse tool bar. In this case, CogniCrypt applies a number of heuristics to determine which project the code should be generated into. First and foremost, if a Java file is opened in the editor when the button is being clicked, CogniCrypt selects its project for code generation. Another way of triggering the code generator is through opening the context menu on the respective project and select the entry "Launch CogniCrypt" as displayed in the screenshot below.
[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/02Context.png|alt=CogniCrypt Context Menu]]

# Selecting a Task

When CogniCrypt launches, the following window pops up:

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/03MainMenu.png|alt=CogniCrypt Main Menu]]

Here, the user first has to select in the upper dropdown menu which project it wants CogniCrypt to generate the code into. The dropdown menu is populated with all Java projects in the workspace. If CogniCrypt was launched by means of the context menu, it auto-selects the respective project. Otherwise, CogniCrypt runs the heuristics sketched above.

The second dropdown menu allows the user to select a cryptographic programming task. Currently, seven tasks are supported. 

The text box, below the second dropdown, shows a brief description of the task that is selected from the dropdown. Once the user has selected both a project and a task they can continue.

# Configuring a Solution
For each task, CogniCrypt asks the user a number of questions. These questions help CogniCrypt configure the solution it provides the user with in the end. For the task ''Encrypt Data Using a Secret Key''(hereafter referred to as Encryption Task), CogniCrypt needs the user to answer the questions shown in the following screenshots:

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/04Questions.png|alt=Questions for Encryption Task]]

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/04Questions1.png|alt=Questions for Encryption Task]]

The first question relates to the required security level as stronger security often impedes the performance of a given solution. Question 2 allows for an easier key generation by giving the user of the generated solution the opportunity to provide a password from which the key is derived. If 'no' is selected, CogniCrypt generates code that employs the regular key generation mechanism of the JCA. If 'yes' is selected, then handling the key should be taken care of by the user himself. The consequence of selecting an answer is specified in the form of ''Note'', below the question. The third question, which appears in the next page, asks if the user has to encrypt large data regularly. The final question allows CogniCrypt's user to specify an input type for the encryption. The default data type are byte arrays. Should the user pick any other type, the generated code takes care of the type conversion for the user.

# Selecting a Solution

When the user has answered all questions, CogniCrypt configures a number of solutions for them. But, the most secure solution is provided to the user as a default solution, as shown in the screenshot below. The preview of the code that will be newly generated is also shown in the same page. If the user has a Java file open, then the code gets generated into the same file. So, the preview also shows the newly generated method inside the user's Java file. If a file is not open then the preview of the new class, that gets created, is shown in the preview. If the user wants to view more algorithm combinations matching his requirements, then the check box below the code preview should be checked and the user can click ''Next''. If not, the user can click ''Finish'' to generate the code in his Java project.

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/05ConfigurationSelection.png|alt=Final Screen for Solution Selection]]

If the user chooses to view other solutions, then a page, as shown in the screenshot below, appears on clicking ''Next''. These algorithm combinations and their variations are ordered by security in descending order, but they are all secure and also fulfill the constraints the user has specified through their answers on the previous page(s). Thus, the user may pick any of the proposed solutions by selecting it in the dropdown menu. CogniCrypt auto-selects the most secure solution (the one which is shown in the previous page) by default. When the user has chosen an algorithm combination from the dropdown, he can view its variations(if any) with the help of ''<'' and ''>'' buttons. The properties of the chosen variation is shown in the ''Instance Details'' panel. They can select a variation of the algorithm combination and hit ''Finish'', which prompts CogniCrypt to generate the solution under the chosen configuration into the user's project.

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/05ConfigurationSelection-Alternatives.png|alt=Final Screen for Solution Selection]]

Additionally, the user can also view the preview of the code for the selected solution, by clicking the button in the same page. This opens a new dialog,as in below screenshot, which presents a comparison between the user's file, before and after the code generation. If no file is open in the user’s end, then a comparison between an empty file and the newly created class file is shown. The dialog shows the code comparison in the same way as that of the ''Compare Editor''
of eclipse. 

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/05ConfigurationSelection-CodeComparison.png|alt=Final Screen for Solution Selection]]

Further, there is also a new wizard that opens on clicking ''Compare Algorithms'' button. This allows the user to compare two algorithms of his choice. This wizard has two combo boxes and two corresponding panels to display the properties of the chosen algorithms. Initially, both the combo boxes have the first element selected by default. Since the elements in the combo boxes are same, their properties will also be identical. If two different algorithm combinations are selected for comparison, then the properties that differ in their values will be highlighted. 

[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/05ConfigurationSelection-AlgorithmComparison.png|alt=Final Screen for Solution Selection]]

# Integrating the Solution into the Application

For the each task, CogniCrypt generates two code artefacts. First, there is the actual implementation. It is always generated into a package called ''Crypto''. The encryption task is rather simple and merely comprises of one class as shown below. Other tasks might be more compley and require more code, though.
[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/07Encryption.png|alt=Actual Implementation of Encryption Task]]

The second artefact is a glue-code method called ''templateUsage'' showcasing to the developer how to properly use the implementation in their application. If, at the time of launching CogniCrypt, there is a Java file open in the Eclipse Editor that also belongs to the project the user selects on CogniCrypt's first screen, CogniCrypt generates the method into  this class assuming that it is the right context for it. Is either of those two conditions not met, i.e., either no Java is opened or the file's project does not match the selected project, CogniCrypt generates the method into a class Output, also under the ''Crypto'' package. 
[[https://github.com/CROSSINGTUD/CogniCrypt/blob/master/documentation/Images%20for%20Tutorial/06TemplateUsage.png|alt=Sample Method showcasing usage of Wrapper Code]]

Finally, to integrate the generated code into their application, the user may choose to simply call the ''templateUsage'' method or copy-paste the statements from the method they need in their code to the right place. By means of its [[code analysis|Code-Analysis]], CogniCrypt will ensure its user does not break the security during integration.