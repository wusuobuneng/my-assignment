# my-assignment
import UIKit

/*
class SecondViewController:UIViewController{
    lazy var start = UILabel()
    ball.frame = CGRect(x: 187.5, y: 506, width: 200, height: 100)
    ball.backgroundColor = #colorLiteral(red: 0.9098039269, green: 0.4784313738, blue: 0.6431372762, alpha: 1)

    var tapRecognizer: UITapGestureRecognizer!
    override func viewDidLoad() {
        super.viewDidLoad()

    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

}
*/
 
class ViewController: UIViewController{
    // 砖块，定义了有几行砖块
    var blocks = [UIView]()
    // 每行砖块数
    let blockCount: Int = 5
    // 小球，懒加载定义小球，即不用这个小球的时候，这个小球就没有作用
    lazy var ball = UIView()
    //懒加载定义开始label
    lazy var startlabel = UILabel()
    // 托盘，懒加载定义托盘
    lazy var pan = UIView()
    // 手势
    var tapRecognizer: UITapGestureRecognizer!              //点击
    var panRecognizer: UIPanGestureRecognizer!              //拖移
    // 托盘初始速度
    var panVelocity: CGFloat = 0
    // 小球的初始速度
    var ballVelocity: CGPoint = CGPoint(x:0, y:0)
    // 定时器；尹讲过，要比NSTimer更好用
    var gameTimer: CADisplayLink?
    //viewDidLoad：加载视图
    
    override func viewDidLoad() {
        super.viewDidLoad();
        setUI()                                             //放入视图
        setGesture()                                        //放入手势
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    //手势
    func setGesture(){
        let tap = UITapGestureRecognizer.init(target: self, action: #selector(ViewController.tap))
        //对点击手势初始化，调用ViewController中的tap的方法
        let panRecognizer = UIPanGestureRecognizer.init(target: self, action: #selector(ViewController.move))
        //对拖移动作初始化，调用ViewController中的move的方法
        self.view.addGestureRecognizer(panRecognizer)       //把panRecognizer手势添加到视图中
        self.view.addGestureRecognizer(tap)                 //把tap手势添加到视图中
    }
    
    // 视图
    func setUI(){
        ball.frame = CGRect(x: 187.5, y: 506, width: 18, height: 18)
        ball.backgroundColor = #colorLiteral(red: 0.9098039269, green: 0.4784313738, blue: 0.6431372762, alpha: 1)
        ball.layer.cornerRadius = ball.frame.width / 2
        ball.center = CGPoint(x:187.5, y:506)
        view.addSubview(ball)
        pan.frame = CGRect(x: 150, y:520, width: 80, height: 10)
        pan.center = CGPoint(x: 187.5, y:520)
        pan.backgroundColor = #colorLiteral(red: 0.1411764771, green: 0.3960784376, blue: 0.5647059083, alpha: 1)
        view.addSubview(pan)
        startlabel.frame = CGRect(x: 100, y: 300 , width: 200, height: 100)
        startlabel.text = "Click here to start"
        view.addSubview(startlabel)
    
        //砖块
        for i in 0...29{
            let colum: Int = i % blockCount
            let row: Int = i / blockCount
            let blockW: CGFloat = view.frame.width / CGFloat(blockCount)
            let blockH: CGFloat = 30
            let blockX: CGFloat = CGFloat(colum) * blockW
            let blockY: CGFloat = CGFloat(row) * blockH
            let block = UIView(frame: CGRect(x: blockX , y: blockY, width: blockW, height: blockH))
            // 砖块儿间隙
            block.layer.borderWidth = 1
            block.layer.borderColor = UIColor.white.cgColor
            block.backgroundColor = #colorLiteral(red: 0.6000000238, green: 0.6000000238, blue: 0.6000000238, alpha: 1)
            view.addSubview(block)
            blocks.append(block)
        }
    }
    
       // 刷新
    func timeUpdate(){
        print("refresh")
        wallHit()
        panHit()
        blockHit()
        failed()
        successed()
        ball.center = CGPoint(x: ball.center.x + ballVelocity.x, y: ball.center.y + ballVelocity.y)
    }
    
    // 开始
    func tap(){
        for block in blocks {
            block.isHidden = false
        }
        self.startlabel.isHidden = true
        ballVelocity = CGPoint(x: -1.5 , y: -1.8)
        gameTimer = CADisplayLink(target: self, selector: #selector(ViewController.timeUpdate))
        gameTimer!.add(to: RunLoop.main, forMode: RunLoopMode.defaultRunLoopMode)
    }
    
    // 托盘的拖移
    func move(sender: AnyObject){
        let panRecognizer = sender as! UIPanGestureRecognizer
        if panRecognizer.state == UIGestureRecognizerState.changed{
            let location = panRecognizer.location(in: self.view)
            self.pan.center = CGPoint(x: location.x, y: self.pan.center.y)
            panVelocity = panRecognizer.velocity(in: self.view).x / 120.0
        }
        if panRecognizer.state == UIGestureRecognizerState.ended{
            panVelocity = 0
        }
    }
    
    // 小球撞到边界，给相应的相反方向速度
    func wallHit(){
        if ball.frame.origin.x < 0.0 {
            ballVelocity = CGPoint(x: abs(ballVelocity.x), y: ballVelocity.y)
        }
        if ball.frame.maxX > UIScreen.main.bounds.size.width {
            ballVelocity = CGPoint(x: -abs(ballVelocity.x), y: ballVelocity.y)
        }
        
        if ball.frame.origin.y < 0.0 {
            ballVelocity = CGPoint(x: ballVelocity.x, y: abs(ballVelocity.y))
        }
    }
    
    // 碰到托盘，给相反的Y速度
    func panHit(){
        if ball.frame.intersects(pan.frame){
            ballVelocity = CGPoint(x: ballVelocity.x + panVelocity, y: -abs(ballVelocity.y))
        }
    }
    
    // 小球打到砖块
    func blockHit(){
        for b in blocks{
             if ball.frame.intersects(b.frame) && b.isHidden == false {
                ballVelocity = CGPoint(x: ballVelocity.x, y: abs(ballVelocity.y))
                b.isHidden = true
            }
        }
    }
    
    // 游戏失败
    func failed() {
        if ball.frame.maxY > UIScreen.main.bounds.size.height {
            startlabel.text = "Sorry，you can try again."
            startlabel.isHidden = false
            gameTimer!.invalidate()
        }
    }
    
    // 游戏胜利
    func successed() {
        var success = true
        for block in blocks {
            if block.isHidden == false {
            success = false
            }
        }
        if success == true {
            //游戏胜利
            gameTimer!.invalidate()
            startlabel.text = "Congratulations ！"
            startlabel.isHidden = false
        }
    }

}
