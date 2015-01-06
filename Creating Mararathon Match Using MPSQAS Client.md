Creating Mararathon Match Using MPSQA Tool
================

This is a guide to teach you how to create marathon matches using `MPSQA Tool`. 

## Creating a contest

1. Run `mpsqas.bat` for Windows or `mpsqas.sh` for Linux. 
2. Login using your credentials. Make sure that `Use SSL` is unchecked.
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_login.png "Login")
3. Click `"Problems" -> "Main Long Problem Room"` in the menu. 
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_problems.png "Problems tab")
4. Click `Create Problem` button. 
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_create_problem.png "Create Problem")
5. Fill in `Class Name`, `Memory Limit (MB)` to 1024 and change `Round Type` into `Long Problem Round`. After that click `Submit`.
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_new_problem.png "New Problem")
6. At this point you will have to wait for the problem to be approved. To check the status, in the menu click `"Problems" -> "Main Long Problem Room"`. 
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_pending_problem.PNG "Pending Problem")

## Approving a Problem (for Admin Role)
1. Login using your credentials. Make sure that `Use SSL` is unchecked.
2. To open pending problems, in the menu click `"Admin" -> "Pending Long Problems"` then you will see a list of `Pending Proposals`. Select a problem in the list then click `View Problem.`
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_pending_proposals.PNG "Pending Proposals")
3. Alternatively, you can just type the problem name on the `Search Box` then click `Go`.
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_search.PNG "Search")
4. With the Problem opened:
  1. Click on the `Admin` tab.
  2. Change the `Status`, for this example we change it to `Final Testing`.
  3. On `Problem Testers`, choose from the `Available Testers` then click `Add`, those testers will be added on the `Scheduled Testers` list.
  4. Click the `Save Changes` button. <strong>* NOTE: NEVER CLICK THE SUBMIT BUTTON. *</strong>
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_problem_approve.png "Problem Approve")

## Modifying and Testing the Approved Problem.
1. In this step, we assume that the problem is already approved. On the menu, click `"Problems -> Main Long Problem Room"`, you will see your created problem already approved(in this example it's in Final Testing).
2. Select the problem and click `View Problem` button.
  * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_problem_final.PNG "Problem Status")
3. In the `Statement Editor` tab: 
  1. Check the `Simple Math` category. Then click `Save Statement` button. <strong>NOTE: If you can't see the checkboxes try resizing the window.</strong>
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_simple_math.png "Simple Math")
  2. Change problem statement part to `Methods`.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_methods.png "Methods")
  3. Modify `Method Name`, `Parameters` and `Returns`, click `Add Method` then click `Save Statement`.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_method_save.png "Modify Method")
  4. Change problem statement part to `Introduction`. Fill the textarea, when you're done click `Save Statement`.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_introduction.PNG "Introductiom")
4. In the `Test Data` tab:
  1. Add new test cases by clicking the `New` button. Try adding 6 Test Cases.
  2. Modify the parameters, in this example our parameter is a String. Set it to "1","2","3","1","2","3". (Don't forget the double quotes).
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_test_string1.png "String Parameter 1")
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_test_string2.png "String Parameter 2")
  3. Check the first 3 test cases as `Example` (Don't mark any test case as `System`). Then click `Save Statement` button.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_test_example.png "Check firs 3 Test Case as Example")
5. In the `Solution` tab:
  1. Enter or paste your code in the editor then click the `Compile` button.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_compile.png "Compile")
  2. You will see the compilation status on the output window.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_compile_status.png "Compile Status")
6. Go back to `Test Data` tab:
  1. Select a test case, then click the `Test` button, you can run the test case on the solution you just submitted.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_test_code.png "Test Solution")
  2. The result will be shown on the output window.
    * ![alt text](https://github.com/macs054/TC-Wiki-Creation/blob/master/img-wiki/marathon-mpsqa/mpsqa_test_output.png "Test Output")
7. Submit the problem by clicking the `Submit` button.






