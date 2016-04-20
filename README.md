#include <stdio.h>
#include <memory.h>
 
#define MAX_LINE 10000
#define MAX_TIMER 5
#define _CLOCKS_PER_SEC 1000
 
static int _clock = 0;
 
inline void clock_f() { _clock += 50; }
inline double getTime() { return 1.0 * _clock / _CLOCKS_PER_SEC; }
 
namespace TIC_TOC {
class timer {
    double _timer[MAX_LINE];  // time for each line
    int _fromLine[MAX_LINE];  // line starting point
    int _lastLineNum;  // mark the ending line
    double _lastRdtsc;  // last time using getime()
    double _totalRdtsc;  // time for all lines
    public:
        timer() {
            memset(_timer, 0, sizeof(_timer));
            memset(_fromLine, 0, sizeof(_fromLine));
            _lastLineNum = -1;
            _lastRdtsc = _totalRdtsc = 0;
        }
        double getAllRdtsc() { return _totalRdtsc; }
        void tic_f(int line) {
            _lastLineNum = line;
            _lastRdtsc = getTime();
        }
        void toc_f(int line) {
            double t = getTime();
            _fromLine[line] = _lastLineNum;
            _timer[line] += t - _lastRdtsc;
            _totalRdtsc += _timer[line];
            _lastLineNum = line;
            _lastRdtsc = t;
        }
        void tictoc(FILE *out) {
            for (int i = 0; i < MAX_LINE; i++)
                if (_timer[i] != 0)
                    fprintf(out, "line %d - %d: %.3fms\n",
                        _fromLine[i], i, _timer[i]);
            fflush(out);
        }
};
class timer_controller {
    int count;  // count timers
    timer timePiece[MAX_TIMER];  // 5 timers at most for each timer_controller
    public:
        timer_controller() { count = 0; }
        void create(int input) { count = input; }
        timer& operator[](const int& input) { return timePiece[input]; }
        void display(FILE *out) {
            for (int i = 0; i < count; i++)
                fprintf(out, "#Timer_%d: %.3fms\n",
                    i+1, timePiece[i].getAllRdtsc());
            fflush(out);
        }
};
}  // namespace TIC_TOC
#include <iostream>
#include "timer.h"
using std::cin;
using std::cout;
using std::endl;
using std::string;
using namespace TIC_TOC;
 
// f(n) = 0, n = 1
// f(n) = 1, n = 2
// f(n) = f(n-1) + f(n-2), n > 2
int fabonaci(int n) {
    clock_f();  // it takes 50 clocks (0.050ms)
    if (n == 1) return 0;
    else if (n == 2) return 1;
    else return fabonaci(n-1) + fabonaci(n-2);
}
 
// f(n) = 1, n = 1
// f(n) = n * f(n-1), n > 1
int factorial(int n) {
    clock_f();
    if (n == 1) return 1;
    return n*factorial(n-1);
}
 
timer_controller T;  // create a timer_controller
int number[5];  // input for test
int count;  // count the lines(tic -> toc)
 
void testTicToc(int n, string function) {
    count = 0;
    cin >> number[0] >> number[1] >> number[2]
        >> number[3] >> number[4];
    for (int i = 0; i < 5; i++) {
        T[n].tic_f(count++);  // tic for starting time
        cout << "input: " << number[i] << ", result: ";
        if (function == "fabonaci")
            cout << fabonaci(number[i]) << endl;
        else if (function == "factorial")
            cout << factorial(number[i]) << endl;
        T[n].toc_f(count);  // toc for ending time
    }
    T[n].tictoc(stdout);  // print out the line-time information
}
 
int main(void) {
    T.create(2);
    testTicToc(0, "fabonaci");
    testTicToc(1, "factorial");
    T.display(stdout);  // print out the total time of each timer
    return 0;
}
 
// Carson's girlfriend helped him design the frame of time.h,
// But if you are good, you should design a better one!
 
/* ------- timer.h -------
static int _clock;
inline void clock_f();
inline double getTime();
 
namespace TIC_TOC {
class timer {
    public:
    timer();
    double getAllRdtsc()
    void tic_f(int);
    void toc_f(int);
    void tictoc(FILE*);
};
class timer_controller {
    timer timePiece[MAX_TIMER];
    public:
    timer_controller();
    void create(int);
    timer& operator[](int&);
    void display(FILE*);
};
}
*/
