# InfixtoPostfix-Calculator
A Calculator that converts an Infix Notation expression into a Postfix Notation expression

#include <stack>
#include <iostream>
using namespace std;

#include <stdexcept>
using std::runtime_error;

string isConverted(string x);
int evalExpression(string x);
bool isParenthesesBal(string x);

class SemanticError: public runtime_error
{
    public:
    SemanticError(): runtime_error("There is a semantic error!"){}
};

string isConverted(string x)
{
   
    string postfix = ""; //holds new expression after being converted to postfix

    
    stack<char> calc; // the stack for the operators

    int n = x.length();
    
   
    for(int i = 0; i < n; i++)
    {	   
        if(x[i] == ' ')
            continue;
        else if(isdigit(x[i])) //if it is an operand output it
        {
		
            postfix += x[i];
            if(!isdigit(x[i+1]))
	    {
	    	postfix += " ";
	    }
	    postfix += " ";
        }
        
        else if(x[i] == '(') //if it is an opening parenthesis push it on the stack
        {
            calc.push(x[i]);
        }
        
        else if(x[i] == '+' || x[i] == '-' || x[i] == '*' || x[i] == '/')
        {
            bool check = true;
            while(check)
            {
                if(x[i] == ' ')
                {
                    continue;
                }
		else if(calc.empty()) // if the stack is empty push the operator
		{
			calc.push(x[i]);
			check = false;
                }
                
                else if(calc.top() == '(') // if the top of the stack is open parenthesis, push the operator 
                {
                    calc.push(x[i]);
                    check = false;
                }
                
                else if((calc.top() == '-' || calc.top() == '+') && (x[i] == '*' || x[i] == '/')) // if it has higher priority than the top of the stack, push operator on stack
                {
                    calc.push(x[i]);
                    
                    check = false;
                }
                
                else // else pop the operator from the stack and output it, and repeat
                {
                    postfix += calc.top();
                    postfix += " ";
                    calc.pop();
                    calc.push(x[i]);
                    check = false;
                }
            }
        }
        
        else if (x[i] == ')') // if it is a closing parenthesis, pop operators from the stack and output them until an opening parenthesis is encounted. Pop and discard the opening parenthesis
        {
            while(calc.top() != '(')
            {
                postfix += calc.top();
                postfix += " ";
                calc.pop();
            }
            if(calc.top() == '(')
            {
                calc.pop();
            }
        }
    }
    
    while(! calc.empty()) // if there are mote inputs, scan for next element, else unstack the remaining operators to output
    {
        postfix += calc.top();
        postfix += " ";
        calc.pop();
    }
    return postfix;
}

int evalExpression(string x)
{
    double op1, op2; // operand1 and operand2 is element popped form the top of the stack
    stack<int> calc; // stack for the operands
    int n = x.length();
    int value;
    string ddigit; //double digit numbers

    for(int i = 0; i < n; i++)
    {
        if(isdigit(x[i])) 
        {
	    if(x[i + 2] == ' ')
            {
                value = int(x[i] - 48);
            }
	    else if(isdigit(x[i+2])) //implementation to read in double digits
	    {
	    	ddigit = ddigit + x[i];
		ddigit = ddigit + x[i+2];
		value = stoi(ddigit);
		i = i + 2;
	    }
            calc.push(value); // if token is a operand push into stack
         
        }
        else if(x[i] == '+') // if token is a operator 
        {
            op2 = calc.top();
            calc.pop();
            op1 = calc.top();
            calc.pop();
            calc.push(op1 + op2); //compute based on operator and push on to stack
        }
        else if(x[i] == '-')
        {
            op2 = calc.top();
            calc.pop();
            op1 = calc.top();
            calc.pop();
            calc.push(op1 - op2);
        }
        else if(x[i] == '*')
        {
            op2 = calc.top();
            calc.pop();
            op1 = calc.top();
            calc.pop();
            calc.push(op1 * op2);
        }
        else if(x[i] == '/')
        {
            op2 = calc.top();
            calc.pop();
            op1 = calc.top();
            calc.pop();
            calc.push(op1 / op2);
        }
    }   
    return calc.top(); //return to top of stack as the result
}

bool isParenthesesBal(string x)
{
	int open = 0; // keeps count of open parentheses
	int closed = 0; // keeps count of closing parentheses
	int n = x.length();

	for(int i = 0; i < n; i++)
	{
		if(x[i] == '(')
		{
			open++;
		}
		else if(x[i] == ')')
		{
			closed++;
		}	
	}
	if(open == closed)
	{
		return true;
	}
	return false;

}

int main()
{
    string infix;
    string postfix;

    cout << "Please type the expression in the format of Infix Notation: " << endl;
    getline(cin, infix); 

    
    for(int i = 0; i < infix.length(); i++)
    {
	// Error 1+2*
        if(infix[infix.length()-1] == '+' || infix[infix.length()-1] == '-' || infix[infix.length()-1] == '*' || infix[infix.length()-1] == '/')
        {
            throw SemanticError();
        }
        // Error 1/0
        if(infix[i] == '/' && infix[i+1] == '0')
        {
            throw SemanticError();
        }
        // Error parentheses do not match 
        if(isParenthesesBal(infix) == false)
        {
            throw SemanticError();
        }
    }
    
    postfix = isConverted(infix); //string postfix used to hold the conversion from infix to post fix
    cout << "The postfix notation: "  << postfix << endl;
    
    cout << "The result : " << evalExpression(postfix) << endl;


    return 0;

}


