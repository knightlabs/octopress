---
layout: post
title: "Notes on Cloud Integration Tests"
date: 2013-11-05 13:07
comments: true
categories: 
---
Been going through collating notes on the IT failures in develop from the weekend: here’s notes on everything _but_ firefox social tests:

0. ~~Forcing router to HTTP1.1 on Nginx fixed the 505s; thx Jim!~~

1. ~~The ‘could not parse content-type’ issue mentioned in Skype on Sunday is gone; Forcing HTTP 1.1 for PIS was apparently bad. Now, Nginx is _only_ forcing HTTP 1.1 on the proxy for registrar and router; Other Nginx configurations remain as they were.~~

2. PIS REST failures seem to be failing at one of fileUpload, testInvalidLearnVersioninPluginFile or makeAPluginWithPostC; in 5 re-runs of build 335, it failed all three the first time, then invalid, make, upload and invalid again. In the past, re-running usually cleared up the one that failed. Running tests again currently (336) to see what happens.

3. ~~Both Chrome Profile _and_ Firefox profile are consistently failing the same 7 tests: testSearchByCourse3-9; FF Profile is also consistently failing 5 tests in the EditProfile test group.~~

4. ~~Chrome Social 2 and 3 seem to be consistently failing testPostNonAsciiCharacters, and testDeleteSpace. FF Social 3 also seems to fail testDeleteSpace, so that might be indicative of an actual issue.~~

5. ~~Sigh. Just watched PIS REST tests run green the first time on 336. So back to square one on figuring out those random failures.~~

6. ~~FF Social 3 seems to have 3 consistent failures across a couple runs: all in the SpaceWallPostsTest~~

7. ~~FF Social 4 seems to have 6 consistent failures across a couple runs: JoinAndLeaveSpaceTest, and the Avatar Upload;~~

8. ~~FF Social 2 seems to have issues with PostingTest, ProfileMergeTest, SpaceCardTest, and FollowPrivateTest:testFollowPrivateProfileFromCard across both build 335 and 336~~

9. ~~FF Social 1: FollowTest and EditProfile Test consistent between 335 and 336.~~

Summarizing: PIS Rest failures appear to remain random; there’s possible actual issues in testSearchByCourse; Chrome may be having issues with non-Ascii characters, and Firefox just still appears to be having random issues, even after Alex moved the webdriver to 2.37 and I confirmed that Xvfb is clean and Firefox launches correctly on all 10 agents.

Hopefully this at least means someone else doesn’t have to do all the correlation.

Bug