# ML_Assignment1

# *პროექტის მიზანი*
Kaggle-ის კონკურსი მიზნად ისახავს სახლის ფასების პრედიქციას 80 ცვლადის საფუძელზე. პროექტში gridsearchCV-თი გატესტილია ორი მოდელი: წრფივი რეგრესია და decision tree და მათ შორის საუკეთესოთი ტესტ სეტზე პრედიქციაა გაკეთებული.

# *რეპოზიტორიის სტრუქტურა*
model_experiment.ipynb - პირველი ნაწილი შეიცავს კლასებს, რომლებიც sklearn pipeline ობიექტისთვის არის გამზადებული. მეორე ნაწილში არის ექსპერიმენტები, რომლებიც ორივე მოდელისთვის საუკეთესო პარამეტრებს არჩევს და mlflow-თი ლოგავს.

model_inference.ipynb - მოიცავს მოდელის საუკეთესო პარამეტრებით დატრენინგებასა და ტესტ სეტზე პრედიქციების დაგენერირებას.

submissions.csv - model_inference.ipynbთი დაგენერირებული პრედიქციები ტესტ სეტზე.

# *Feature engineering*
NaN მნიშვნელობების შევსებისას რიცხვით ცვლადებში ნულების ჩაწერა შეგვიძლია და ეს აზრობრივადაც სწორი მიდგომაა, რადგან missing რიცხვითი ცვლადები მიემართება ან რაიმეს ფართობს, ან ოთახების(/აივნების/გარაჟების...) რაოდენობებს. მონაცემების გადახედვისას ჩანს, რომ missing კატეგორიული ცვლადები სწორედ ამ რიცხვით ცვლადებთან ერთად გვხვდება, ამიტომ მათ ადგილზე ჩაწერილია No_{category_name}.

კატეგორიული ცვლადების რიცხვითად გადაყვანის ნაწილში გამოყენებულია მხოლოდ one-hot encoder, რადგან WOE encoding უწყვეტი თარგეთის პრედიქციისას არ გამოდგება ბინებად დაყოფის გარეშე. 

ვინაიდან მხოლოდ 1500მდე მონაცემი გვაქვს, მონაცემების გადაყრა არც ისე სახარბიელო იქნებოდა. ამასთანავე, ფასის განაწილებაში მკვეთრი აუთლაიერები არ შეინიშნება, შესაბამისად ყველა სახლზე მონაცემს ვიტოვებთ:
<img width="769" alt="Screenshot 2025-04-07 at 2 34 07 pm" src="https://github.com/user-attachments/assets/36a87935-adf7-4bec-97e6-de340592d96c" />

# *Feature selection*
Feature selection ნაწილში გამოყენებულია RFE. წრფივი რეგრესიისთვის estimator არის LinearRegressor, ხოლო decision tree-სთვის DecisionTreeRegressor. 

# *Training*
პროექტში გატესტილია ორი მოდელი: წრფივი რეგრესია და decision tree შემდეგი ჰიპერპარამეტრების გრიდებით:

param_grid_lin = {
    'scaler': scalers,
    'feature_selector__n_features': [10, 20, 50, 100, 150],
}

param_grid_tree = {
    'scaler': [StandardScaler(), MinMaxScaler(), None],
    'feature_selector__n_features': [10, 20, 50, 150],
    'regressor__min_samples_leaf': [3, 5, 10],
    'regressor__min_samples_split': [5, 10, 20],
    'regressor__max_depth': [5, 10, 20],
}

ტრენინგის ნაწილში ვუყურებთ მოდელების პერფორმანსს RMSE, MAE და R^2 მეტრიკების საშუალებით. GridsearchCV ახდენს მოდელების ოპტიმიზირებას R^2 პარამეტრზე დაყრდნობით, ხოლო RMSE და MAE გამოყენებულია იმისთვის, რომ მოდელის პერფორმანსი ადამიანისთვის მარტივად გასაგები და ინტუიტიური ფორმითაც გვქონდეს წარმოდგენილი.
მონაცემთა სიმცირის გამო gridsearch-ში სტანდარტული ქროს-ვალიდაციის ნაცვლად გამოყენებულია 4-fold cross validation. უნდა აღვნიშნო, რომ კოდის გასამარტივებლად მონაცემთა შევსებას და ენკოდინგს ფაიფლაინში train/test სპლიტამდე ვაკეთებ, მაგრამ ამ შევსების ლოგიკების დასაწერად მხოლოდ ტრენინგ სეტი მქონდა გამოყენებული და ტესტ სეტზე bias არ გაჩნდებოდა. 

საუკეთესო წრფივ მოდელს train-test სეტებზე R^2 ჰქონდა 0.819 და 0.749, ხოლო MAE 24322$ და 28067$. MAE-ებს შორის სხვაობა ზედმეტად დიდი არ არის, ანუ მოდელი overfit-ზე არ გადის. ასევე არ არის underfit, რადგან train/test სეტებზე ასიათასობით დოლარიანი სახლებისთვის ზომიერი ~4000$იანი სხვაობა გვაქვს.

საუკეთესო ხის მოდელს R^2 აქვს 0.856 და 0.81, ხოლო MAE 17964$ და 23177$. აქაც ანალოგიურად არ გვაქვს მკვეთრი overfit და MAE-ებს შორის 5000$იანი სხვაობაც წინა მოდელთან ახლოს არის.

საბოლოოდ უნახავი ტესტ სეტისთვის ავირჩიე ხის მოდელი მისი შედარებით უკეთესი R^2ის გამო, რადგან ის დატასეტში ვარიაციის უფრო დიდ წილს აღწერს.
საუკეთესო პარამეტრები არჩეული მოდელისთვის:
best_parameters = {'feature_selector__n_features': 10, 
                   'regressor__max_depth': 20, 
                   'regressor__min_samples_leaf': 10, 
                   'regressor__min_samples_split': 20, 
                   'scaler': None}


# *MLFlow tracking*
წრფივი რეგრესიის ექსპერიმენტი:
https://dagshub.com/dimna21/ML_Assignment1/experiments#/experiment/m_c1e8dd3f70c64bf68caf87b5383b6d1b

ხის მოდელის ექსპერიმენტი:
https://dagshub.com/dimna21/ML_Assignment1/experiments#/experiment/m_7702289a9828469bad55790437f20e5b

MLFlow-თი ჩაწერილია training ნაწილში მოცემული dictionary-ს პარამეტრების სივრცე თითოეული მოდელისთვის და დალოგილია RMSE, MAE და R^2 მეტრიკები train/test სეტებისთვის.

საუკეთესო ხის მოდელმა Kaggle competition-ის ტესტ სეტზე აჩვენა 0.18204 RMSLE. იმის გათვალისწინებით, რომ leaderboardის თავში ხამები არიან, რომლებიც test set-ზე overfitting-ში ხარჯავენ თავისუფალ დროს და 0.00044 აქვთ ერორი, ამ შედარებით მარტივი არქიტექტურის მოდელებით მიღებული 0.18 ნორმალური შედეგია.

