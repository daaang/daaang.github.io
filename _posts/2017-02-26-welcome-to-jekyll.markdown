---
layout: post
title:  "Changing this first post"
date:   2017-02-26 18:43:19 -0500
---
This isn't what this post used to look like. *History makes liars of us
all.* Never believe anything; never trust anyone.

Alright here's some code

{% highlight fortran %}
      PROGRAM EULER
        INTEGER, PARAMETER  :: MX = 9999, MXC = 10000, COLS = 72
        INTEGER             :: DNMTR, CLN,
     1                         NMRTR,
     2                         AI, A(2, MXC),
     3                         LENS(MX),
     4                         LCM,
     5                         I, J, K,
     6                         LDC
        DNMTR     = 1
        CLN       = 0
        LENS      = -1
        LENS(1)   = 0
        LDC       = 0
        DO 70 I = 2, MX
          IF ( LENS(I) .NE. -1 ) GOTO 70
          LDC = LDC + 1
          AI = 0
          NMRTR = 10
   10     IF ( AI .NE. MXC ) GOTO 15
          WRITE (6, '(A, I4)') 'I breached the maximum cycle length and
     1had to stop at 1 / ', I
          STOP
   15     DO 20 J = 1, AI
            IF ( A(1, J) .NE. NMRTR .OR.
     1           A(2, J) .NE. I ) GOTO 20
            LENS(I) = AI - J
            IF ( CLN .GE. LENS(I) ) GOTO 40
            DNMTR = I
            CLN = LENS(I)
            GOTO 40
   20     CONTINUE
          AI = AI + 1
          A(1, AI) = NMRTR
          A(2, AI) = I
          NMRTR = 10 * MOD(NMRTR, I)
          IF ( NMRTR .GT. 0 ) GOTO 10
          LENS(I) = 0
          DO 30 J = 2, MX / I
            IF ( LENS(J) .EQ. -1 ) GOTO 30
            LENS( I * J ) = LENS(J)
   30     CONTINUE
          GOTO 70
   40     DO 60 J = 2, MX / I
            IF ( LENS(J) .EQ. -1 ) GOTO 60
            IF ( LENS(J) .EQ. 0 ) GOTO 50
            LENS( I * J ) = LCM( LENS(I), LENS(J) )
            IF ( LENS( I * J ) .LE. CLN ) GOTO 60
            DNMTR = I * J
            CLN = LENS( I * J )
            GOTO 60
   50       LENS( I * J ) = LENS(I)
   60     CONTINUE
   70   CONTINUE
        AI = 0
        NMRTR = 10
   80   DO 150 I = 1, AI
          IF ( A(1, I) .NE. NMRTR .OR.
     1         A(2, I) .NE. DNMTR ) GOTO 150
          WRITE (6, 900, ADVANCE='NO') DNMTR
          K = 12
          DO 100 J = 1, I-1
            IF ( K .GE. COLS ) GOTO 90
            WRITE (6, '(I1)', ADVANCE='NO') A(1, J) / A(2, J)
            K = K + 1
            GOTO 100
   90       WRITE (6, '(I1)') A(1, J) / A(2, J)
            WRITE (6, '(A)', ADVANCE='NO') '  '
            K = 2
  100     CONTINUE
          IF ( K .GE. COLS ) GOTO 110
          WRITE (6, '(A)', ADVANCE='NO') '('
          K = K + 1
          GOTO 120
  110     WRITE (6, '(I1)') A(1, J) / A(2, J)
          WRITE (6, '(A)', ADVANCE='NO') '  '
          K = 2
  120     DO 140 J = I, AI
            IF ( K .GE. COLS ) GOTO 130
            WRITE (6, '(I1)', ADVANCE='NO') A(1, J) / A(2, J)
            K = K + 1
            GOTO 140
  130       WRITE (6, '(I1)') A(1, J) / A(2, J)
            WRITE (6, '(A)', ADVANCE='NO') '  '
            K = 2
  140     CONTINUE
          WRITE (6, '(A)') ')'
          WRITE (6, '(A, I5, A)') 'The cycle length is: ', CLN, ' digits
     1.'
          WRITE (6, '(A, I4, A)') 'Long division was run ', LDC, ' times
     1.'
          STOP
  150   CONTINUE
        AI = AI + 1
        A(1, AI) = NMRTR
        A(2, AI) = DNMTR
        NMRTR = 10 * MOD(NMRTR, DNMTR)
        GOTO 80
  900 FORMAT ('1 / ', I4, ' = 0.')
      END
{% endhighlight %}

you're welome
