# Home-Banner
This class shows how to create banner in objective 

//
//  HomeViewController.m
//  AlEnna
//
//  Created by Mehboob on 10/23/18.
//  Copyright © 2018 360Nautica. All rights reserved.
//

#import "HomeViewController.h"
#import "RegisterVC.h"
#import "LoginVC.h"
@interface HomeViewController ()<UIGestureRecognizerDelegate,UIScrollViewDelegate>


@property (weak, nonatomic) IBOutlet UIActivityIndicatorView *bannerActivity;
@property (weak, nonatomic) IBOutlet UIView *banner;
@property(strong,nonatomic)NSMutableArray *bannersArray;
@property(strong,nonatomic)NSMutableArray *bannerImages;
@property(strong,nonatomic)NSArray *bannerArray;



@end

@implementation HomeViewController{
    
    
    BOOL buttonClick;
    
    
    UIPageControl *pageController;
    
    int nextPage;
    UITapGestureRecognizer *Banner_Tap;
    
    int imageTag;
    UIImageView *bannerImageView;
    
    CGFloat screenWidth;
    
}

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
 
    
    [self initialSetup];
}

-(void)initialSetup{
       self.navigationController.navigationBar.tintColor = [UIColor whiteColor];
    [self updateViewWithLanguage];
    self.bannersArray = [[NSMutableArray alloc] init];
    self.bannerImages = [[NSMutableArray alloc] init];
    
    [self getBanners];
}

-(void)getBanners{
    [self.bannerActivity startAnimating];
    self.bannerArray = [[NSArray alloc] init];
    [self startActivityIndicatorWithActivity:NSLocalizedString(@"AtPik_Loading", @"")];
    [[WebServiceManager sharedInstance] getRequestWithEndpoint:GET_HOME_PAGE_API successCallback:^(NSDictionary *successDictionary, NSError *error) {
        [self stopActivityIndicator];
      //  if ([[successDictionary valueForKey:@"Status"] intValue] == kAPI_SUCCESS){
            
            NSLog(@"%@",successDictionary);
            self.bannerArray = [[successDictionary valueForKey:@"homebanner"] valueForKey:@"imageurl"];
            
            NSLog(@"%@",self.bannerArray);
            NSLog(@"%@",[[successDictionary valueForKey:@"homebanner"] valueForKey:@"imageurl"]);
            
            for (NSString *imageString in [[successDictionary valueForKey:@"homebanner"] valueForKey:@"imageurl"]) {
                //                UIImage *bannerImage = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:imageString]]];
                //                //Added recently
                //                if (bannerImage != nil) {
                //                    [self.bannerImages addObject:bannerImage];
                //                }
                
                
                dispatch_async(dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
                    //Background Thread
                    UIImage *bannerImage = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:imageString]]];
                    if (bannerImage != nil) {
                        [self.bannerImages addObject:bannerImage];
                    }
                    dispatch_async(dispatch_get_main_queue(), ^(void){
                        //Run UI Updates
                        
                        if ([[[successDictionary valueForKey:@"homebanner"] valueForKey:@"imageurl"] count] == [self.bannerImages count]) {
                            [self.bannerActivity stopAnimating];
                            self.bannerActivity.hidden = YES;
                            [self bannerSetup];
                        }
                        
                        NSLog(@"Banner Setup");
                    });
                    
                });
                
                
                
            }
            
            //  [self bannerSetup];
            
            
     //   }
        
        NSLog(@"%@",self.bannerImages);
        
    } failiureCallBack:^(NSDictionary *failuerDict, NSError *error) {
        [self stopActivityIndicator];
        [self showAlertMessage:nil title:error.localizedDescription ok:NSLocalizedString(@"OK_Title", @"") cancel:nil];
    }];
}
-(void)bannerSetup{
    UIScrollView *scr=[[UIScrollView alloc] initWithFrame:CGRectMake(0, 0, self.banner.frame.size.width, _banner.frame.size.height)];
    scr.tag = 1;
    scr.autoresizingMask=UIViewAutoresizingNone;
    scr.showsHorizontalScrollIndicator = NO;
    scr.showsVerticalScrollIndicator = NO;
    [self.banner addSubview:scr];
    scr.pagingEnabled = YES;
    
    [self setupScrollView:scr];
    scr.delegate = self;
    [pageController setTag:12];
    pageController.numberOfPages=self.bannerImages.count;
    pageController.autoresizingMask=UIViewAutoresizingNone;
    
    pageController.pageIndicatorTintColor = [UIColor greenColor];
    pageController.currentPageIndicatorTintColor = [UIColor redColor];
    [_banner addSubview:pageController];
   
}
- (void)setupScrollView:(UIScrollView*)scrMain {
    // we have 10 images here.
    // we will add all images into a scrollView &amp; set the appropriate size.
    
    if (self.bannerImages.count > 0) {
        
        
        for (int i=0; i<self.bannerImages.count; i++) {
            // create image
            UIImage *image = [self.bannerImages objectAtIndex:i];
            // create imageView
            bannerImageView = [[UIImageView alloc] initWithFrame:CGRectMake((i)*scrMain.frame.size.width, 0, scrMain.frame.size.width, scrMain.frame.size.height)];
            // set scale to fill
            bannerImageView.contentMode=UIViewContentModeScaleToFill;
            // set image
            [bannerImageView setImage:image];
            // apply tag to access in future
            // bannerImageView.tag=i;
            bannerImageView.userInteractionEnabled=YES;
            Banner_Tap=[[UITapGestureRecognizer alloc] init];
            [Banner_Tap addTarget:self action:@selector(Banner_Click:)];
            bannerImageView.tag=i;
            [bannerImageView addGestureRecognizer:Banner_Tap];
            
            // add to scrollView
            [scrMain addSubview:bannerImageView];
        }
    }
    // set the content size to 10 image width
    [scrMain setContentSize:CGSizeMake(scrMain.frame.size.width*self.bannerImages.count, scrMain.frame.size.height)];
    // enable timer after each 2 seconds for scrolling.
    [NSTimer scheduledTimerWithTimeInterval:3 target:self selector:@selector(scrollingTimer) userInfo:nil repeats:YES];
}

- (void)scrollingTimer {
    // access the scroll view with the tag
    UIScrollView *scrMain = (UIScrollView*) [_banner viewWithTag:1];
    // same way, access pagecontroll access
    UIPageControl *pageControl = (UIPageControl*) [_banner viewWithTag:12];
    pageControl.pageIndicatorTintColor = [UIColor greenColor];
    pageControl.currentPageIndicatorTintColor = [UIColor redColor];
    // get the current offset ( which page is being displayed )
    CGFloat contentOffset = scrMain.contentOffset.x;
    // calculate next page to display
    nextPage = (int)(contentOffset/scrMain.frame.size.width) + 1 ;
    // if page is not 10, display it
    if( nextPage!=self.bannerImages.count )  {
        [scrMain scrollRectToVisible:CGRectMake(nextPage*scrMain.frame.size.width, 0, scrMain.frame.size.width, scrMain.frame.size.height) animated:YES];
        pageControl.currentPage=nextPage;
        // else start sliding form 1 :)
    } else {
        [scrMain scrollRectToVisible:CGRectMake(0, 0, scrMain.frame.size.width, scrMain.frame.size.height) animated:YES];
        pageControl.currentPage=0;
    }
}
- (void)Banner_Click:(UITapGestureRecognizer *)gesture {
    UIView *tap_View = gesture.view;
    
    NSLog(@"%ld",tap_View.tag);
    
    
//    RegisterVC *vc = [self.storyboard instantiateViewControllerWithIdentifier:kRegisterVCID];
//
//    [self.navigationController pushViewController:vc animated:YES];
    
    LoginVC *vc = [self.storyboard instantiateViewControllerWithIdentifier:kLoginVCID];
    
    [self.navigationController pushViewController:vc animated:YES];
    
    
    
}
- (IBAction)selctLanguage:(id)sender {
    
    
    [self performSegueWithIdentifier:@"LanguageVC" sender:self];
}


-(void)updateViewWithLanguage{
    
    //    self.whatAreYouLookingOutlet.text = NSLocalizedString(@"What_are_you_looking_for", @"");
    //    self.tpDealsOutlet.text = NSLocalizedString(@"Top_Deals_For_You", @"");
    //    self.browseByCatOutlet.text = NSLocalizedString(@"Browse_By_Category", @"");
    //    self.recomendationForYouOutlet.text = NSLocalizedString(@"Recommendation_For_You", @"");
    //    self.brandsOutlet.text = NSLocalizedString(@"Brands", @"");
    //    self.superMarketOutlet.text = NSLocalizedString(@"Super_Market", @"");
    //    self.babyOutlet.text = NSLocalizedString(@"Baby", @"");
    //    [self.browseByCategorySeeAll setTitle:NSLocalizedString(@"See_All", @"") forState:UIControlStateNormal] ;
    //    [self.recomendedForYouSeeAll setTitle:NSLocalizedString(@"See_All", @"") forState:UIControlStateNormal] ;
    //    [self.superMarketSeeAll setTitle:NSLocalizedString(@"See_All", @"") forState:UIControlStateNormal] ;
    //    [self.babySeeAll setTitle:NSLocalizedString(@"See_All", @"") forState:UIControlStateNormal] ;
    
    
    NSString * language = [[NSUserDefaults standardUserDefaults] valueForKey:@"currentLanguageKey"];
    if ([language isEqualToString:@"ar"]) {
        NSLog(@"user's prefered first language is Arabi");
       
         [UIView appearance].semanticContentAttribute = UISemanticContentAttributeForceLeftToRight;
       // self.whatAreYouLookingOutlet.textAlignment = NSTextAlignmentRight;
       
        
    }else{
      
    
         [UIView appearance].semanticContentAttribute = UISemanticContentAttributeForceRightToLeft;
          //  self.whatAreYouLookingOutlet.textAlignment = NSTextAlignmentLeft;
        
    }
    
    
    
}
//Note : Take UIView and banner as outlet

@end
