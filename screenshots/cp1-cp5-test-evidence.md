# CP1-CP5 test evidence

Date: 2026-09-29 (Asia/Saigon)
Command: `python -X utf8 -m pytest tests/test_cp1.py tests/test_cp2.py tests/test_cp3.py tests/test_cp4.py tests/test_cp5.py -q -rs`
Docker Desktop CLI was available on PATH for CP2 build tests.

........................................................................ [ 86%]
.......ssss                                                              [100%]
=========================== short test summary info ===========================
SKIPPED [1] tests\test_cp5.py:179: Chỉ chạy khi LOCAL_FALLBACK=true
SKIPPED [1] tests\test_cp5.py:183: Chỉ chạy khi LOCAL_FALLBACK=true
SKIPPED [1] tests\test_cp5.py:186: Chỉ chạy khi LOCAL_FALLBACK=true
SKIPPED [1] tests\test_cp5.py:190: Chỉ chạy khi LOCAL_FALLBACK=true
79 passed, 4 skipped in 6.75s

Final grade command: `python -X utf8 grade.py --no-bonus`
Result: required work 100/100, with CP1 13/13, CP2 16/16, CP3 22/22,
CP4 19/19, CP5 live deployment 9/9, and exercises 10/10.
The four CP5 skips are only LOCAL_FALLBACK checks; they do not count as passes
or add points. The nine live Railway tests all passed.
